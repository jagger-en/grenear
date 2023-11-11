<script setup>
import Notification from './components/Notification.vue'
import LineChart from './components/LineChart.vue'
import SmallIcon from './components/icons/smallIcon.vue';

import firebaseConfig from "../firebaseInit";
import { initializeApp } from "firebase/app";
import { getFirestore, collection, getDocs, addDoc, deleteDoc, doc } from 'firebase/firestore'
</script>

<script>
const app = initializeApp(firebaseConfig)
const fs = getFirestore(app)
export default {
  data() {
    return {
      notifications: [],
      responses_groups: [],
      selected_subscriber_id: '',
      responses: [],
      subscribers: []
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
  <main class="centering-mobile">
    <div class="container-mobile">
      <div class="container mt-3">
        <LineChart />
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
