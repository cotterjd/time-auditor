<template>
  <ul v-for="dateKey in Object.keys(groupedActivities)" :key="dateKey" class="activity-list">
    <span>{{ dateKey }}</span>
    <li v-for="activity in groupedActivities[dateKey]" :key="activity.id">
      {{ activity.start }}-{{ activity.finish }}({{ activity.timeTook }}) {{ activity.activity }}
    </li>
  </ul>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  name: `ActivityList`,
  components: {},
  props: [`groupedActivities`],
})
</script>

<style scoped>
.activity-list {
  list-style: none;
  text-align: left;
}
</style>
