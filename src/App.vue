<template>
  <div class="container">

    <h1>Smart Home</h1>

    <div>
      <span :class="['dot', { connected: isConnected }]"></span>
      <span>{{ isConnected ? "Connected" : "Disconnected" }}</span>
    </div>

    <div class="rooms">

      <!-- Wohnzimmer -->
      <div class="room">
        <h2>Living Room</h2>

        <Temperature 
          :temperature="temperature1"
          :time="tempTime1"
        />

        <LedControl 
          :ledStatus="ledStatus1"
          :ledBusy="ledBusy1"
          @led-command="(cmd) => sendLedCommand(1, cmd)"
        />
      </div>

      <!-- Schlafzimmer -->
      <div class="room">
        <h2>Bedroom</h2>

        <Temperature 
          :temperature="temperature2"
          :time="tempTime2"
        />

        <LedControl 
          :ledStatus="ledStatus2"
          :ledBusy="ledBusy2"
          @led-command="(cmd) => sendLedCommand(2, cmd)"
        />
      </div>

    </div>

  </div>
</template>

<script>
import mqtt from "mqtt"
import Temperature from "./components/Temperature.vue"
import LedControl from "./components/LedControl.vue"

export default {
  components: { Temperature, LedControl },

  data() {
    return {
      client: null,
      isConnected: false,

      // Wohnzimmer
      temperature1: "--",
      tempTime1: "-",
      ledStatus1: "UNKNOWN",
      ledBusy1: false,

      // Schlafzimmer
      temperature2: "--",
      tempTime2: "-",
      ledStatus2: "UNKNOWN",
      ledBusy2: false
    }
  },

  mounted() {
    this.connectMQTT()
  },

  methods: {

    connectMQTT() {
      this.client = mqtt.connect("ws://192.168.2.239:9001")

      this.client.on("connect", () => {
        this.isConnected = true

        this.client.subscribe("home/temperature1")
        this.client.subscribe("home/temperature2")
        this.client.subscribe("home/led1/status")
        this.client.subscribe("home/led2/status")
      })

      this.client.on("message", (topic, message) => {
        const payload = message.toString()

        if (topic === "home/temperature1") {
          this.temperature1 = Number(payload).toFixed(1)
          this.tempTime1 = new Date().toLocaleTimeString()
        }

        if (topic === "home/temperature2") {
          this.temperature2 = Number(payload).toFixed(1)
          this.tempTime2 = new Date().toLocaleTimeString()
        }

        if (topic === "home/led1/status") {
          this.ledBusy1 = false
          this.ledStatus1 = payload
        }

        if (topic === "home/led2/status") {
          this.ledBusy2 = false
          this.ledStatus2 = payload
        }
      })

      this.client.on("close", () => {
        this.isConnected = false
      })
    },

    sendLedCommand(node, cmd) {

      const topicSet = `home/led${node}/set`

      if (node === 1 && this.ledBusy1) return
      if (node === 2 && this.ledBusy2) return

      if (node === 1) {
        this.ledBusy1 = true
        this.ledStatus1 = "WAITING"
      }

      if (node === 2) {
        this.ledBusy2 = true
        this.ledStatus2 = "WAITING"
      }

      this.client.publish(topicSet, cmd, { qos: 1 })

      let retries = 0

      const interval = setInterval(() => {

        if ((node === 1 && !this.ledBusy1) || (node === 2 && !this.ledBusy2)) {
          clearInterval(interval)
          return
        }

        // this.client.publish(topicSet, "LED:State", { qos: 1 })

        retries++

        if (retries >= 5) {
          clearInterval(interval)

          if (node === 1) {
            this.ledBusy1 = false
            this.ledStatus1 = "ERROR"
          }

          if (node === 2) {
            this.ledBusy2 = false
            this.ledStatus2 = "ERROR"
          }
        }

      }, 500)
    }
  }
}
</script>
