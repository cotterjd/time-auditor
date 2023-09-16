<template>
  <div class="header-container">
    <p><i>No more wondering where the day went</i></p>
    <button v-if="!start" @click="startDay">Start Day</button>

    <div v-show="start">
      <input v-model="activity" @keyup.enter="submitActivity" />
      <button @click="onSubmit">Submit</button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({
  name: `Header`,
  components: {},
  data: () => ({
    activity: ``,
  }),
  props: [`start`, `submitActivity`],
  methods: {
    onSubmit() {
      this.$emit(`submitActivity`, this.activity)
      this.activity = ``
    }
  },
})
</script>

<style scoped>
.header-container {
  text-align: center
}
</style>
