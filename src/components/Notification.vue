<script setup>
defineProps({
  msg: {
    type: String,
    required: true
  }
})
</script>

<template>
  <div>
    <div>Message: {{ msg }}</div>
  </div>
</template>
