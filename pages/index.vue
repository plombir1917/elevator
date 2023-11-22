<template>
  <section>
    <h1>{{ newSession }}</h1>
  </section>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'IndexPage',
  data: () => ({
    newSession: [],
  }),

  async mounted() {
    this.newSession = await this.$axios.$post(
      'https://lstu.fusionsoft.ru/srv-elevator/newSession?timeFactor=1.0&floors=3&clientName=захар'
    )
  },
})
</script>
