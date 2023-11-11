<script setup>
import SmallIcon from './icons/smallIcon.vue'
</script>

<script>
export default {
  name: 'Notification',
  props: {
    notification: {
      type: Object,
      required: true
    }
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
  <div class="card">
    <div class="card-header">Call for action!</div>
    <div class="card-body">
      <p class="card-text">{{ notification.msg }}</p>
      <div class="row">
        <div class="col">
          <button class="btn btn-success" @click="acceptChallenge()">I'm in</button>
        </div>
        <div class="col">
          <div class="btn btn-danger" @click="rejectChallenge()">Nah!</div>
        </div>
      </div>
    </div>
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
