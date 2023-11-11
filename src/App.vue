<script setup>
import Notification from './components/Notification.vue'

import firebaseConfig from "../firebaseInit";
import { initializeApp } from "firebase/app";
import { getFirestore, collection, getDocs } from "firebase/firestore";
</script>

<script>
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
      const app = initializeApp(firebaseConfig);
      const fs = getFirestore(app)
      const collection_ref = collection(fs, "notifications")

      const notifications = []
      await getDocs(collection_ref).then((snapshot) => {
        snapshot.docs.forEach(d => {
          notifications.push(d.data())
        })
      })

      return notifications
    }
  }
}
</script>

<template>
  <main>
    <h1>Hello World</h1>
    <li v-for="notification in notifications">
      {{ notification.msg }}
    </li>
    <Notification msg="This is the notification." />
  </main>
</template>
