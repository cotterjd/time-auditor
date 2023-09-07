<template>
  <div id="home-container">
    <p><i>No more wondering where the day went</i></p>
    <button v-if="!start" @click="startDay">Start Day</button>

    <div v-show="start">
      <input v-model="activity" @keyup.enter="submitActivity" />
      <button @click="submitActivity">Submit</button>
    </div>
  </div>

  <ul class="activity-list">
    <li v-for="activity in activities" :key="activity.id">
      {{ activity.start }}-{{ activity.finish }}({{ activity.timeTook }}) {{ activity.activity }}
    </li>
  </ul>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import { DateTimeFormatOptions } from '../types'

interface Activity {
  start: string
  finish: string
  activity: string
}
interface Data {
  start: Date | null
  activity: string
  activities: Activity[]
}

const timeOptions: DateTimeFormatOptions = {
  hour12: false,
  hour: `2-digit`,
  minute: `2-digit`,
}
export default defineComponent({
  name: `Home`,
  components: {},
  data: (): Data => ({
    start: null,
    activity: ``,
    activities: [],
  }),
  computed: {},
  watch: {},
  methods: {
    submitActivity() {
      if (!this.start) return
      const startTime: string = this.start?.toLocaleTimeString([], timeOptions)
      const endTime: string = new Date().toLocaleTimeString([], timeOptions)
      const newActivity = {
        start: startTime,
        finish: endTime,
        activity: this.activity,
        timeTook: Math.floor(((new Date()).getTime() - this.start.getTime()) / 1000 / 60),
      }
      this.activities = [newActivity, ...this.activities]
      this.start = new Date()
      this.activity = ``
    },
    startDay() {
      this.start = new Date()
    },
  },
})

</script>

<style scoped>

.activity-list {
  list-style: none;
  text-align: left;
}

#home-container {
  text-align: center
}

</style>
