<template>
  <div class="card">
    <h2>Temperature</h2>
    <div class="big">{{ temperature }} °C</div>
    <div>Update: {{ time }}</div>
  </div>
</template>

<script>
export default {
  props: ["temperature", "time"]
}
</script>