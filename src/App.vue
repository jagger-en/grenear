<script setup>
import Notification from './components/Notification.vue'
import SmallIcon from './components/icons/smallIcon.vue';

import firebaseConfig from "../firebaseInit";
import { initializeApp } from "firebase/app";
import { getFirestore, collection, getDocs,
         addDoc, deleteDoc, doc } from "firebase/firestore";
</script>

<script>
const app = initializeApp(firebaseConfig);
const fs = getFirestore(app)
export default {
  data() {
    return {
      notifications: [],
      responses_groups: [],
      selected_subscriber: null,
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
      const responses_groups = this.notifications.map(n => {
        const responses_in = responses.filter(resp => resp.response_type_id == 'rkRK3v2aW8jEBrA2ZvLa' && resp.notification_id == n.id)
        const subscriber_ids_in = responses_in.map(r => {
          return r.subscriber_id
        })
        const subscribers_in = this.subscribers.filter(s => subscriber_ids_in.includes(s.id))
        return {
          notification: n,
          subscribers_in: subscribers_in
        }
      })
      this.responses_groups = responses_groups
    })
  },
  methods: {
    async getNotifications() {
      return this.getDocs("notifications")
    },
    async getSubscribers() {
      return this.getDocs("subscribers")
    },
    async getResponses() {
      return this.getDocs("responses")
    },
    async getDocs(collection_id) {
      const collection_ref = collection(fs, collection_id)
      const docs = []
      await getDocs(collection_ref).then((snapshot) => {
        snapshot.docs.forEach(d => {
          docs.push({id: d.id, ...d.data()})
        })
      })
      return docs
    },
    async acceptChallenge(notification_id) {
      const collection_ref = collection(fs, "responses")
      const response = {'timestamp': this.getCurrentTimestamp(),
                        'notification_id': notification_id,
                        'subscriber_id': this.selected_subscriber,
                        'response_type_id': 'rkRK3v2aW8jEBrA2ZvLa'}
      await addDoc(collection_ref, response)
      window.location.reload()
    },
    async deleteRecord(collection_id, record_id) {
      const docRef = doc(fs, collection_id, record_id)
      await deleteDoc(docRef)
      window.location.reload()
    },
    getCurrentTimestamp() {
      return new Date()
    }
  }
}
</script>

<template>
  <main>
    <nav class="navbar navbar-expand-lg bg-body-tertiary">
      <div class="container-fluid">
        <a class="navbar-brand" href="#">
          <SmallIcon class="brand-icon" />
          One2Line
        </a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarSupportedContent">
          <label>Logged in as:</label>
          <select v-model="selected_subscriber" class="form-control">
            <option :value=subscriber.id v-for="subscriber in subscribers">{{ subscriber.first_name }} {{ subscriber.last_name }}</option>
          </select>
        </div>
      </div>
    </nav>

    <div class="p-2">
      <Notification v-for="notification in notifications"
        class="notification" :notification=notification
        @deleteRecord="deleteRecord"
        @acceptChallenge="acceptChallenge" />
    </div>

    <div v-for="responses_group in responses_groups">
      <h3>Message: {{ responses_group.notification.msg }}</h3>
      <table v-if="responses_group.subscribers_in.length > 0">
        <thead>
          <tr>
            <th>Name</th>
            <th>City</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="subscriber in responses_group.subscribers_in">
            <td>{{ subscriber.first_name }} {{ subscriber.last_name }}</td>
            <td>{{ subscriber.city_name }}</td>
          </tr>
        </tbody>
      </table>

    </div>
</main>
</template>

<style>
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
}
</style>