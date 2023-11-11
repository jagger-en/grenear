<script setup>
import Notification from './components/Notification.vue'
import LineChart from './components/LineChart.vue'
import SmallIcon from './components/icons/smallIcon.vue';
import PlusIcon from './components/icons/plusIcon.vue';

import firebaseConfig from "../firebaseInit";
import { initializeApp } from "firebase/app";
import { getFirestore, collection, getDocs, addDoc } from 'firebase/firestore'
</script>

<script>
const app = initializeApp(firebaseConfig)
const fs = getFirestore(app)
export default {
  data() {
    const todayPretty = '2023-11-04' // hardcoded for demo purposes
    const today = new Date(todayPretty)
    const tomorrow = new Date(todayPretty)
    tomorrow.setHours(tomorrow.getHours() + 24)
    return {
      notifications: [],
      responses_groups: [],
      selected_subscriber_id: '',
      responses: [],
      subscribers: [],
      todayPretty,
      today,
      tomorrow,
      goForItShown: false
    }
  },
  mounted() {
    this.getNotifications().then((n) => {
      this.notifications = n
    })

    this.getSubscribers().then((s) => {
      this.subscribers = s
    })

    this.getResponses().then((responses) => {
      const responses_groups = this.notifications.map((n) => {
        const responses_in = responses.filter(
          (resp) => resp.response_type_id == 'rkRK3v2aW8jEBrA2ZvLa' && resp.notification_id == n.id
        )
        const subscriber_ids_in = responses_in.map((r) => {
          return r.subscriber_id
        })
        const responses_out = responses.filter(
          (resp) => resp.response_type_id == 'Tg7XLemrnn5BKc33MvrQ' && resp.notification_id == n.id
        )
        const subscriber_ids_out = responses_out.map((r) => {
          return r.subscriber_id
        })
        const subscribers_in = this.subscribers.filter((s) => subscriber_ids_in.includes(s.id))
        const subscribers_out = this.subscribers.filter((s) => subscriber_ids_out.includes(s.id))

        const subscribers_in_out = [
          ...subscribers_in.map((d) => {
            return { ...d, in_out: 'IN' }
          }),
          ...subscribers_out.map((d) => {
            return { ...d, in_out: 'OUT' }
          })
        ]
        return {
          notification: n,
          subscribers_in_out: subscribers_in_out
        }
      })
      this.responses = responses
      this.responses_groups = responses_groups
    })
  },
  methods: {
    async getNotifications() {
      return this.getDocs('notifications')
    },
    async getSubscribers() {
      return this.getDocs('subscribers')
    },
    async getResponses() {
      return this.getDocs('responses')
    },
    async getDocs(collection_id) {
      const collection_ref = collection(fs, collection_id)
      const docs = []
      await getDocs(collection_ref).then((snapshot) => {
        snapshot.docs.forEach((d) => {
          docs.push({ id: d.id, ...d.data() })
        })
      })
      return docs
    },
    async acceptChallenge(notification_id) {
      const collection_ref = collection(fs, 'responses')
      const response = {
        timestamp: this.getCurrentTimestamp(),
        notification_id: notification_id,
        subscriber_id: this.selected_subscriber_id,
        response_type_id: 'rkRK3v2aW8jEBrA2ZvLa'
      }
      await addDoc(collection_ref, response)
      window.location.reload()
    },
    async rejectChallenge(notification_id) {
      const collection_ref = collection(fs, 'responses')
      const response = {
        timestamp: this.getCurrentTimestamp(),
        notification_id: notification_id,
        subscriber_id: this.selected_subscriber_id,
        response_type_id: 'Tg7XLemrnn5BKc33MvrQ'
      }

      const matches = this.responses.filter(
        (r) =>
          r.notification_id == notification_id &&
          r.subscriber_id == this.selected_subscriber_id &&
          r.response_type_id == 'Tg7XLemrnn5BKc33MvrQ'
      )
      if (matches.length == 0) {
        await addDoc(collection_ref, response)
      }
      this.notifications = this.notifications.filter((n) => n.id != notification_id)
    },
    getCurrentTimestamp() {
      return new Date()
    },
    toggleGoForIt() {
      this.goForItShown = !this.goForItShown
    }
  }
}
</script>

<template>
  <nav class="navbar navbar-expand-lg navbar-light bg-light centering-mobile">
    <div class="container-mobile">
      <a class="navbar-brand" href="#">
        <SmallIcon class="brand-icon" />
        One2Line
      </a>
      <div class="logged-in-as">
        <label class="me-2">Logged in as:</label>
        <select v-model="selected_subscriber_id" class="user-select">
          <option disabled value="">Please select one</option>
          <option :value=subscriber.id v-for="subscriber in subscribers">{{ subscriber.first_name }} {{ subscriber.last_name }}</option>
        </select>
      </div>
    </div>
  </nav>
  <main class="centering-mobile mt-5">
    <div class="container-mobile">

      <div class="forecast border-rounded mb-3">
        <div class="forecast-header p-3">
          <div class="forecast-header-text">CLEAN ENERGY FORECAST</div>
          <div>{{ todayPretty }}</div>
        </div>
        <div class="mt-3">
          <LineChart :today="today" :tomorrow="tomorrow" />
        </div>
        <div class="simple-text mt-3 p-3">
          Green shading indicates sufficient clean energy,
          while red shading highlights its deficiency.
        </div>
      </div>


      <div class="how-can-i-help mt-5">
        <h5>How can I help with the case?</h5>
        <p class="simple-text">
          By estimating the time you reduce your electricity consumption
          during peak hours, we can calculate your contribution to the
          common goal of sustainable electricity usage!
        </p>
      </div>


      <div class="go-for-it border-rounded mb-3 no-padding">
        <div class="go-for-it-header">
          <div>Go for it!</div>
          <div class="plus-icon-wrapper" @click="toggleGoForIt()">
            <PlusIcon class="plus-icon" />
          </div>
        </div>
        <div v-if="goForItShown" class="go-for-it-contents border-rounded">

          <div class="card" style="font-size: 0.9em; margin-bottom:1em;">
            <div class="card-body" style="padding: 0.7em !important;">
              <div class="row align-items-center">
                <div class="col-5">
                  <div class="row">
                    <p class="card-title no-margin">Electric device</p>
                  </div>
                  <div class="row">
                    <p class="card-text text-body-secondary">Wattage</p>
                  </div>
                </div>
                <div class="col-7">
                  <div class="row">
                    <div class="col no-margin-no-padding">
                    </div>
                    <div class="col-5 no-padding">
                      <div class="input-group align-items-center">
                        <input
                          type="text"
                          class="form-control"
                          placeholder="min"
                          aria-label="minute"
                          aria-describedby="basic-addon1"
                        />
                      </div>
                    </div>
                    <div class="col no-margin-no-padding">
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="card" style="font-size: 0.9em; margin-bottom:2%;">
            <div class="card-body">
              <div class="row align-items-center">
                <div class="col-5">
                  <div class="row">
                    <p class="card-title no-margin">Electric device</p>
                  </div>
                  <div class="row">
                    <p class="card-text text-body-secondary">Wattage</p>
                  </div>
                </div>
                <div class="col-7">
                  <div class="row">
                    <div class="col no-margin-no-padding">
                    </div>
                    <div class="col-5 no-padding">
                      <div class="input-group align-items-center">
                        <input
                          type="text"
                          class="form-control"
                          placeholder="min"
                          aria-label="minute"
                          aria-describedby="basic-addon1"
                        />
                      </div>
                    </div>
                    <div class="col no-margin-no-padding">
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="text-center">
              <a class="simple-text mt-3 show-more" href="#">Show more</a>
          </div>

        </div>
      </div>

      <div class="row" style="margin: 1% 0;">
          <div class="col-8 no-padding">
              <p class="simple-text mt-3">By joining the challenge you can save</p>
          </div>
          <div class="col simple-text mt-3" style="padding-right: 0;">
              {counter} euros
          </div>
      </div>

      <div class="row align-items-center" style="margin: 5% 0;">
          <div class="col"></div>
          <button class="col-6 btn tone-green simple-text">JOIN THE CHALLENGE!</button>
          <div class="col"></div>
      </div>


      <div class="row">
          <div class="col-5" style="padding-right: 0; margin-right: 1em !important;">
            <div class="card">
              <div class="card-body text-center">
                <p class="card-title">{total_joined}</p>
                <p class="card-text simple-text">people have already joined</p>
              </div>
            </div>
          </div>
          <div class="col no-margin-no-padding">
            <div class="card">
              <div class="card-body">
                  <div class="row">
                    <div class="col">
                    </div>
                    <div class="col">
                    </div>
                  </div>
                  <div class="row">
                    <div class="progress" role="progressbar" aria-label="Basic example" aria-valuenow="37" aria-valuemin="0" aria-valuemax="100">
                      <div class="progress-bar tone-green" style="width: 37%"></div>
                    </div>
              </div>
              </div>
            </div>
          </div>
      </div>

      <!-- Button trigger modal -->
      <button type="button" class="btn tone-green" data-bs-toggle="modal" data-bs-target="#exampleModal">TODO event trigger by timer</button>

      <!-- Modal -->

      <div class="modal modal-dialog modal-dialog-centered fade" id="exampleModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header">
              <h1 class="modal-title fs-5" id="exampleModalLabel">Modal title</h1>
              <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <div class="modal-body">
              ...
            </div>
            <div class="modal-footer">
              <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
              <button type="button" class="btn btn-primary">Save changes</button>
            </div>
          </div>
        </div>
      </div>

      <div class="container text-center">
        <div class="row">
          <Notification
            v-for="notification in notifications"
            class="notification"
            :notification="notification"
            @rejectChallenge="rejectChallenge"
            @acceptChallenge="acceptChallenge"
          />
        </div>
      </div>

      <div class="container">
        <div v-for="responses_group in responses_groups">
          <h3>Message: {{ responses_group.notification.msg }}</h3>
          <table class="table" v-if="responses_group.subscribers_in_out.length > 0">
            <thead>
              <tr class="table-info">
                <th>Name</th>
                <th>City</th>
                <th>IN/OUT</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="subscriber in responses_group.subscribers_in_out">
                <td>{{ subscriber.first_name }} {{ subscriber.last_name }}</td>
                <td>{{ subscriber.city_name }}</td>
                <td>{{ subscriber.in_out }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </main>
</template>

<style>

.go-for-it {
  border-radius: 7px;
  background: #f2ebfffc;
  display: flex;
  flex-wrap: wrap;
}

.go-for-it-header {
  color: #40007e;
  display: flex;
  width: 100%;
  justify-content: space-between;
  padding: 15px 20px;
}

.go-for-it-contents {
  color: #40007e;
  width: 100%;
  padding: 15px 20px;
}

.plus-icon-wrapper {
  width: 25px;
  height: 25px;
  border-radius: 50px;
  background-color: #fff;
  display: flex;
  justify-content: center;
  align-items: center;
  align-content: center;
  cursor: pointer;
}

.plus-icon-wrapper:hover {
  background-color: #c2c0c0;
}

.plus-icon {
  width: 17px;
  height: 17px;
}

.how-can-i-help  {
  text-align: center;
}

.how-can-i-help h5 {
  font-size: 14px;
}

.simple-text {
  font-size: 12px;
  line-height: 15px;
  font-weight: 400;
}

.border-rounded {
  border: 1px solid #d4d4d4b7;
  border-radius: 7px;
  /* padding: 7px 10px; */
  box-shadow: 0px 5px 5px 0px #0000002d;
}

.forecast-header {
  font-size: 12px;
  display: flex;
  line-height: 15px;
  justify-content: space-between;
}

.forecast-header-text {
  font-weight: 600;
  width: 140px;
}

.container-mobile {
  width: 320px;
}

.centering-mobile {
  display: flex;
  justify-content: center;
}


.user-select {
  width: 200px;
}

.logged-in-as {
  display: flex;
  justify-content: space-between;
}

.brand-icon {
  margin-right: 5px;
}

.brand {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  align-content: center;
  font-size: 20px;
}

.notification {
  margin-bottom: 7px;
  max-width: 400px;
}
</style>
