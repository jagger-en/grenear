<script setup>
import Notification from './components/Notification.vue'

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
  },
  methods: {
    async getNotifications() {
      const collection_ref = collection(fs, "notifications")
      const notifications = []
      await getDocs(collection_ref).then((snapshot) => {
        snapshot.docs.forEach(d => {
          notifications.push({id: d.id, ...d.data()})
        })
      })
      return notifications
    },
    async getSubscribers() {
      const collection_ref = collection(fs, "subscribers")
      const subscribers = []
      await getDocs(collection_ref).then((snapshot) => {
        snapshot.docs.forEach(d => {
          subscribers.push({id: d.id, ...d.data()})
        })
      })
      return subscribers
    },
    async addNotification() {
      const collection_ref = collection(fs, "notifications")
      await addDoc(collection_ref, {'msg': 'foo-bar'})
      window.location.reload()
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
    <h1>Hello World</h1>
    <select v-model="selected_subscriber" name="" id="">
      <option :value=subscriber.id v-for="subscriber in subscribers">{{ subscriber.first_name }} {{ subscriber.last_name }}</option>
    </select>
    <div v-for="notification in notifications">
      <Notification class="notification" :notification=notification
        @deleteRecord="deleteRecord"
        @acceptChallenge="acceptChallenge" />
    </div>
    <button @click="addNotification()">Add new</button>
</main>
</template>

<style>
main {
  padding: 15px 20px;
}

.notification {
  margin-bottom: 7px;
}
</style>