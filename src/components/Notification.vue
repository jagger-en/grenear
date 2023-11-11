<script setup>
import SmallIcon from './icons/smallIcon.vue'
</script>

<script>

export default {
  name: 'Notification',
  props: {
    notification: {
          type: Object,
          required: true,
      },
  },
  emits: ['rejectChallenge', 'acceptChallenge'],
  methods: {
    rejectChallenge() {
      this.$emit('rejectChallenge', this.notification.id)
    },
    acceptChallenge() {
      this.$emit('acceptChallenge', this.notification.id)
    }
  }
}

</script>

<template>
  <div class="notification">
    <div class="header">
      <div class="header-left">
        <SmallIcon class="small-icon" />
        <span>One2Line</span>
      </div>
      <div class="header-right">
        <div @click="rejectChallenge()">&times</div>
      </div>
    </div>
    <div class="title">Call to action</div>
    <div class="msg">Message: {{ notification.msg }}</div>
    <button @click="acceptChallenge()">I'm in</button>
  </div>
</template>

<style scoped>
.notification {
  border: 1px solid rgb(189, 189, 189);
  border-radius: 5px;
  padding: 5px 10px;
  background: #fff;
}
.notification .title {
  font-weight: 600;
}

.header {
  display: flex;
  justify-content: space-between;
}

.header-left {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  align-content: center;
}

.header-right {
  display: flex;
  justify-content: flex-start;
  align-items: flex-start;
  align-content: flex-start;
  font-size: 20px;
  cursor: pointer;
}

.small-icon {
  margin-right: 5px;
}

.header span {
  color: #969191;
  font-weight: 500;
}

</style>