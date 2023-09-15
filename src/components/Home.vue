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
  <p>Version: {{ version }}</p>
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
  version: string
  db: IDBDatabase | null
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
    version: require(`../../package.json`).version,
    db: null, 
  }),
  computed: {},
  watch: {},
  created() {
    const openDBRequest = window.indexedDB.open(`timeLogs`, 1)
    openDBRequest.onupgradeneeded = (evt: any) => {
      const db = evt.target.result
      if (!db) {
        console.error(`Error opening indexedDB`)
        return
      }
      const objectStore = db.createObjectStore(`activities`, { autoIncrement: true })
      if (!objectStore) {
        console.error(`Error creating object objectStore`)
        return
      }
      objectStore.createIndex(`start`, `start`, { unique: false })
      objectStore.createIndex(`finish`, `finish`, { unique: false })
      objectStore.createIndex(`activity`, `activity`, { unique: false })

      this.db = db
    }
    openDBRequest.onsuccess = (evt: any) => {
      const db = evt.target.result
      this.db = db
    }
  },
  mounted() {
    this.start = localStorage.getItem(`start`) ? new Date(localStorage.getItem(`start`) as string) : null

    const openDBRequest = window.indexedDB.open(`timeLogs`, 1)
    openDBRequest.onsuccess = (evt: any) => {
      const db = evt.target.result
      const objectStore = db.transaction([`activities`], `readwrite`).objectStore(`activities`)
      this.db = db

      objectStore.openCursor().onsuccess = (event: any) => {
        const cursor = event?.target?.result
        if (cursor) {
          this.activities.unshift(cursor.value)
          cursor.continue()
        }
      }
    }
  },
  methods: {
    submitActivity() {
      if (!this.db) return
      if (!this.start) return

      const startTime: string = this.start?.toLocaleTimeString([], timeOptions)
      const endTime: string = new Date().toLocaleTimeString([], timeOptions)
      const newActivity = {
        start: startTime,
        finish: endTime,
        activity: this.activity,
        timeTook: this.getTimeTook(),
      }
      const objectStore = this.db.transaction([`activities`], `readwrite`).objectStore(`activities`)
      const addRequest = objectStore.add(newActivity)
      addRequest.onsuccess = (evt: Event) => {
        this.activities = [newActivity, ...this.activities]
        localStorage.setItem(`start`, new Date().toISOString())
        this.start = new Date()
        this.activity = ``
      }
      addRequest.onerror = (evt: Event) => {
        console.error(`Error adding activity`, evt)
      }
    },
    startDay() {
      localStorage.setItem(`start`, new Date().toISOString())
      this.start = new Date()
    },
    getTimeTook() {
      if (!this.start) return ``
      const minutes = Math.floor(((new Date()).getTime() - this.start.getTime()) / 1000 / 60)
      return `${Math.floor(minutes / 60)}:${minutes % 60}`
    }
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
