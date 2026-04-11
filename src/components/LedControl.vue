<template>
  <div class="card">
    <h2>LED</h2>

    <button 
      class="on"
      :disabled="ledBusy"
      @click="$emit('led-command', 'ON')">
      ON
    </button>

    <button 
      class="off"
      :disabled="ledBusy"
      @click="$emit('led-command', 'OFF')">
      OFF
    </button>

    <p>
      Status: 
      <span :style="statusStyle">
        {{ ledStatus }}
      </span>
    </p>
  </div>
</template>

<script>
export default {
  props: ["ledStatus", "ledBusy"],

  computed: {
    statusStyle() {
      if (this.ledStatus === "ON") return { color: "lightgreen" }
      if (this.ledStatus === "OFF") return { color: "red" }
      if (this.ledStatus === "WAITING") return { color: "orange" }
      return { color: "gray" }
    }
  }
}
</script>