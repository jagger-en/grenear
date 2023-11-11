<template>
    <div class="chart-wrapper">
        <div class="chart-scroll">
            <Line id="my-chart-id" :options="chartOptions" :data="chartData" />
        </div>
    </div>
</template>

<script>
import { Line } from 'vue-chartjs'
import {
    Chart as ChartJS,
    CategoryScale,
    LinearScale,
    PointElement,
    LineElement,
    Title,
    Tooltip,
    Legend,
    Filler
} from 'chart.js'


ChartJS.register(
    CategoryScale,
    LinearScale,
    PointElement,
    LineElement,
    Title,
    Tooltip,
    Legend,
    Filler
)

export default {
    name: 'LineChart',
    components: { Line },
    data() {
        return {
            start_time: '2023-11-11T00:00:00Z',
            end_time: '2023-11-12T00:00:00Z',
            chartData: {
                labels: [],
                datasets: [
                    {
                        label: 'Data One',
                        backgroundColor: '#f87979',
                        data: [],
                        fill: {
                            target: 'origin',
                            above: 'rgb(0,255,0)',
                            below: 'rgb(255,0,0)'
                        }
                    }
                ]
            },
            chartOptions: {
                plugins: {
                    legend: {
                        display: false
                    },
                },
                elements: {
                    point: {
                        radius: 0
                    }
                }
            }
        }
    },
    mounted() {
        this.fetchSurplusDeficitData()
    },
    methods: {
        createListOfConsecutiveDatesByHours(starting, ending) {
            const start = new Date(starting) // copy the date object
            const end = new Date(ending)
            const hours_diff = (end - start) / (60 * 60 * 1000)
            const listOfDates = [new Date(start)]
            for (var i = 1; i <= hours_diff; i++) {
                const date = start.setHours(start.getHours() + 1)
                listOfDates.push(new Date(date))
            }
            return listOfDates
        },
        fetchSurplusDeficitData() {
            fetch(`https://api.fingrid.fi/v1/variable/186/events/json?start_time=${this.start_time}&end_time=${this.end_time}`).then((response) => {
                response.json().then((data) => {
                    if (response.ok) {
                        const times = data.map(d => d.start_time) // ignore end_time

                        const starting_time = times[0]
                        const ending_time = times[times.length-1]

                        const dates = this.createListOfConsecutiveDatesByHours(starting_time, ending_time)

                        const chart_data = dates.map(dt => {
                            const next_dt = new Date(dt)
                            next_dt.setHours(next_dt.getHours() + 1)

                            const matches = data.filter(d => new Date(d.start_time) <= next_dt && new Date(d.start_time) >= dt)
                            const total_for_hour = matches.reduce((acc, d) => acc + d.value, 0)
                            return total_for_hour
                        })
                        const chart_labels = dates.map(d => d.getHours())
                        this.chartData = {
                            labels: chart_labels,
                            datasets: [
                                {
                                    label: 'Surplus and Deficit',
                                    backgroundColor: '#f87979',
                                    data: chart_data,
                                    fill: {
                                        target: 'origin',
                                        above: '#00ff82',
                                        below: '#ef555cfc'
                                    }
                                }
                            ]
                        }
                    }
                })
            })
        }
    }
}
</script>

<style>
#my-chart-id {
    background: #fff;
    width: 2000px;
}

.chart-wrapper {
    overflow: auto;
}

.chart-scroll {
    width: fit-content;
    height: 200px;
}
</style>
