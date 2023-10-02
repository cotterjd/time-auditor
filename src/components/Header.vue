<template>
  <div class="header-container">
    <p><i>No more wondering where the day went</i></p>
    <button class="btn start-btn" v-if="!start" @click="startTimer">Start Day</button>

    <div class="input-container" v-show="start">
      <input class="input" v-model="activity" @keyup.enter="submitActivity" />
      <button id="submit-button" class="btn" @click="onSubmit">
        Submit Finished Activity
      </button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue"

export default defineComponent({
  name: `Header`,
  components: {},
  data: () => ({
    activity: ``,
  }),
  props: [`start`, `submitActivity`, `startDay`],
  methods: {
    onSubmit() {
      this.$emit(`submitActivity`, this.activity)
      this.activity = ``
    },
    startTimer() {
      this.startDay()
    },
  },
})
</script>

<style scoped>
.header-container {
  text-align: center;
}
.input {
  width: 300px;
  height: 30px;
  border-radius: 5px;
  border: 1px solid #ccc;
  padding: 5px;
  margin: 5px;
  font-size: 16px;
}
.input-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.btn {
  width: 310px;
  border-radius: 5px;
  border: 1px solid #ccc;
  padding: 10px 0;
  margin: 5px;
  font-size: 16px;
  background-color: #49d6db;
  color: #000;
}
.start-btn {
  height: 80px;
  font-size: 20px;
}
</style>
