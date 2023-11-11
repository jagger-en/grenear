<template>
    <div>
        <div class="row mb-2">
            <div class="col-6">
                <label>Start time</label>
                <input v-model="start_time" @change="fetchSurplusDeficitData()" type="text" class="form-control">
            </div>
            <div class="col-6">
                <label>End time</label>
                <input v-model="end_time" @change="fetchSurplusDeficitData()" type="text" class="form-control">
            </div>
        </div>
        <Line id="my-chart-id" :options="chartOptions" :data="chartData" />
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
            start_time: '2023-11-01T00:00:00Z',
            end_time: '2023-11-02T00:00:00Z',
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
                responsive: true,
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
        fetchSurplusDeficitData() {
            fetch(`https://api.fingrid.fi/v1/variable/186/events/json?start_time=${this.start_time}&end_time=${this.end_time}`).then((response) => {
                response.json().then((data) => {
                    if (response.ok) {
                        const chart_labels = data.map(d => d.start_time)
                        const chart_data = data.map(d => d.value)
                        this.chartData = {
                            labels: chart_labels,
                            datasets: [
                                {
                                    label: 'Surplus and Deficit',
                                    backgroundColor: '#f87979',
                                    data: chart_data,
                                    fill: {
                                        target: 'origin',
                                        above: 'rgb(0,255,0)',
                                        below: 'rgb(255,0,0)'
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
}
</style>
