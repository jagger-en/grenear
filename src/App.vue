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
const collection_ref = collection(fs, "notifications")
export default {
  data() {
    return {
      notifications: []
    }
  },
  mounted() {
    this.getNotifications().then((n) => {
      this.notifications = n
    })
  },
  methods: {
    async getNotifications() {

      const notifications = []
      await getDocs(collection_ref).then((snapshot) => {
        snapshot.docs.forEach(d => {
          notifications.push({id: d.id, ...d.data()})
        })
      })

      return notifications
    },
    async addNotification() {
      await addDoc(collection_ref, {'msg': 'foo-bar'})
      window.location.reload()
    },
    async deleteRecord(collection_id, record_id) {
      const docRef = doc(fs, collection_id, record_id)
      await deleteDoc(docRef)
      window.location.reload()
    }
  }
}
</script>

<template>
  <main>
    <h1>Hello World</h1>
    <div v-for="notification in notifications">
      <Notification class="notification" :notification=notification
        @deleteRecord="deleteRecord" />
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