<template>
  <div class="header">
    <button @click="goForward">&lt;</button>
    <button @click="goBack">&gt;</button>
  </div>
  <span>{{ Object.keys(groupedActivities)[index] }}</span>
  <div
    v-for="activity in groupedActivities[Object.keys(groupedActivities)[index]]"
    :key="activity.id"
    class="activity-list"
  >
    <span class="activity-name">{{ activity.activity }}</span>
    <span class="activity-span"
      >{{ activity.start }}-{{ activity.finish }}</span
    >
    <span class="activity-time">{{ activity.timeTook }}</span>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  name: `ActivityList`,
  components: {},
  props: [`groupedActivities`, `goBack`, `goForward`],
  data() {
    return {
      index: 0,
    }
  },
})
</script>

<style scoped>
.activity-list {
  text-align: left;
  display: grid;
  grid-template-areas:
    "name name name time"
    "span span span time";
  border: 1px solid gray;
  border-radius: 5px;
  padding: 5px;
}
.activity-name {
  grid-area: name;
}
.activity-span {
  grid-area: span;
}
.activity-time {
  grid-area: time;
  text-align: end;
  display: grid;
  align-items: center;
}
.header button {
  padding: 10px;
  width: 50%;
}
</style>
