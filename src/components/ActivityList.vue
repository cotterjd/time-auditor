<template>
  <span>{{ Object.keys(groupedActivities)[index] }}</span>
  <div v-for="activity in groupedActivities[Object.keys(groupedActivities)[index]]" :key="activity.id">
    {{ activity.start }}-{{ activity.finish }}({{ activity.timeTook }}) {{ activity.activity }}
  </div>
  <div class="footer">
    <button @click="previous">&lt;</button>
    <button @click="next">&gt;</button>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  name: `ActivityList`,
  components: {},
  props: [`groupedActivities`],
  data() {
    return {
      index: 0,
    }
  },
  methods: {
    previous() {
      this.index--
    },
    next() {
      this.index++
    },
  },
})
</script>

<style scoped>
.activity-list {
  list-style: none;
  text-align: left;
  padding: 0 10px;
}
.footer button {
  padding: 10px;
  width: 50%;
}
</style>
