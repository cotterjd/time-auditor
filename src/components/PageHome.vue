<template>
  <ta-header :start="start" @submitActivity="submitActivity" @aggregateActivity="aggregateActivity" :startDay="startDay" />

  <activity-list :groupedActivities="groupedActivities" :goBack="goBackADay" :goForward="goForwardADay" />

    <p>{{this.aggHours}}:{{this.aggMinutes}} of {{this.aggActivity}} in the past week</p>   
  <version />
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import Version from './Version.vue'
import TaHeader from './Header.vue'
import ActivityList from './ActivityList.vue'
import { DateTimeFormatOptions } from '../types'
import { groupBy } from 'ramda'
import { sub, subWeeks, differenceInDays } from 'date-fns'

interface Activity {
  start: string
  finish: string
  activity: string
  date: string
}
interface Data {
  start: Date | null
  activities: Activity[]
  db: IDBDatabase | null
  groupedActivities: any
  aggActivity: string
  aggHours: number
  aggMinutes: number
  day: number
}

const timeOptions: DateTimeFormatOptions = {
  hour12: false,
  hour: `2-digit`,
  minute: `2-digit`,
}

export default defineComponent({
  name: `Home`,
  components: {
    Version,
    TaHeader,
    ActivityList,
  },
  data: (): Data => ({
    start: null,
    activities: [],
    groupedActivities: {},
    db: null, 
    aggActivity: ``,
    aggHours: 0,
    aggMinutes: 0,
    day: 0,
  }),
  computed: {},
  watch: {
    activities (newVal) {
      this.groupedActivities = groupBy((activity: Activity) => activity.date, newVal)
    },
    day (newVal) {
      // reset activities to fetch the switched-to day's activities
      this.activities = []
      const objectStore = this.db.transaction([`activities`], `readwrite`).objectStore(`activities`)

      objectStore.openCursor().onsuccess = (event) => {
        const cursor = event?.target?.result
        if (cursor) {
          if (cursor.value.date.includes(sub(new Date(), { days: this.day }).toLocaleDateString())) {
            this.activities = [cursor.value, ...this.activities]
          }
          cursor.continue()
        }
      }
    }
  },
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
      objectStore.createIndex(`date`, `date`, { unique: false })

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

      objectStore.openCursor().onsuccess = (event) => {
        const cursor = event?.target?.result
        if (cursor) {
          if (cursor.value.date.includes(new Date().toLocaleDateString())) {
            this.activities = [cursor.value, ...this.activities]
          }
          cursor.continue()
        }
      }
    }
  },
  methods: {
    submitActivity(activity: string) {
      if (!this.db) return
      if (!this.start) return

      const startTime: string = this.start?.toLocaleTimeString([], timeOptions)
      const endTime: string = new Date().toLocaleTimeString([], timeOptions)
      const newActivity = {
        start: startTime,
        finish: endTime,
        activity: activity,
        timeTook: this.getTimeTook(),
        date: `${(new Date()).toString().split(` `)[0]} ${new Date().toLocaleDateString()}`,
      }
      const objectStore = this.db.transaction([`activities`], `readwrite`).objectStore(`activities`)
      const addRequest = objectStore.add(newActivity)
      addRequest.onsuccess = (evt: Event) => {
        this.activities = [newActivity, ...this.activities]
        localStorage.setItem(`start`, new Date().toISOString())
        this.start = new Date()
      }
      addRequest.onerror = (evt: Event) => {
        console.error(`Error adding activity`, evt)
      }

    },
    startDay() {
      console.log(`start day`)
      localStorage.setItem(`start`, new Date().toISOString())
      this.start = new Date()
    },
    getTimeTook() {
      if (!this.start) return ``
      const minutes = Math.floor(((new Date()).getTime() - this.start.getTime()) / 1000 / 60)
      return `${Math.floor(minutes / 60)}:${minutes % 60}`
    },
    goBackADay() {
      // looks counter-intuitive but day is being used in a subtract date function
      this.day++
    },
    goForwardADay() {
      this.day--
    },
    aggregateActivity(activity: string) {
      this.aggActivity = activity
      this.aggHours = 0
      this.aggMinutes = 0
      const objectStore = this.db.transaction([`activities`], `readwrite`).objectStore(`activities`)
      objectStore.openCursor(null, `prev`).onsuccess = (event) => {
        const cursor = event?.target?.result
        if (cursor) {
          if (isOverAWeekOld(new Date(cursor.value.date))) {
            return
          }
          console.log(cursor.value.date)
          if (cursor.value.activity === activity) {
            const [hours, minutes] = cursor.value.timeTook.split(`:`) 
            this.aggHours = this.aggHours + Number(hours)
            this.aggMinutes = this.aggMinutes + Number(minutes)
            if (this.aggMinutes >= 60) {
              this.aggMinues = this.aggMinutes - 60
              this.aggHours = this.aggHours + 1
            }
          }
          cursor.continue()
        }
      }
    }
  },
})

function isOverAWeekOld(date: Date): boolean {
  const sevenDaysAgo = subWeeks(new Date(), 1);
  return differenceInDays(new Date(), date) > 7;
}

</script>

<style scoped>



</style>
