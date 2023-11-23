<template>
  <section>
    <div>
      <b-navbar toggleable="sm" type="light" variant="light">
        <b-navbar-toggle target="nav-text-collapse"></b-navbar-toggle>

        <b-navbar-brand>{{ newSession.sessionId }}</b-navbar-brand>

        <b-collapse id="nav-text-collapse" is-nav>
          <b-navbar-nav>
            <b-nav-text>{{ newSession.closedSessionId }}</b-nav-text>
          </b-navbar-nav>
        </b-collapse>
      </b-navbar>
    </div>
    <div class="form">
      <b-form @submit="onSubmit" @reset="onReset" v-if="show">
        <b-form-group
          id="input-group-3"
          label="Быстро вверх:"
          label-for="input-3"
        >
          <b-form-select
            id="input-3"
            v-model="nextCycle.MoveUpFast"
            :options="options"
            required
          ></b-form-select>
        </b-form-group>

        <b-form-group
          id="input-group-3"
          label="Медленно вверх:"
          label-for="input-3"
        >
          <b-form-select
            id="input-3"
            v-model="nextCycle.MoveUpSlow"
            :options="options"
            required
          ></b-form-select>
        </b-form-group>

        <b-form-group
          id="input-group-3"
          label="Быстро вниз:"
          label-for="input-3"
        >
          <b-form-select
            id="input-3"
            v-model="nextCycle.MoveDownFast"
            :options="options"
            required
          ></b-form-select>
        </b-form-group>

        <b-form-group
          id="input-group-3"
          label="Медленно вниз:"
          label-for="input-3"
        >
          <b-form-select
            id="input-3"
            v-model="nextCycle.MoveDownSlow"
            :options="options"
            required
          ></b-form-select>
        </b-form-group>

        <b-form-group
          id="input-group-3"
          label="Закрыть дверь:"
          label-for="input-3"
        >
          <b-form-select
            id="input-3"
            v-model="nextCycle.DoClose"
            :options="options"
            required
          ></b-form-select>
        </b-form-group>

        <b-form-group
          id="input-group-3"
          label="Открыть дверь:"
          label-for="input-3"
        >
          <b-form-select
            id="input-3"
            v-model="nextCycle.DoOpen"
            :options="options"
            required
          ></b-form-select>
        </b-form-group>

        <b-button type="submit" variant="primary">Отправить</b-button>
        <b-button type="reset" variant="danger">Сбросить</b-button>
      </b-form>
    </div>
    <b-button
      variant="outline-primary"
      v-if="newSession.sessionId"
      class="newCycle_button"
      @click.prevent="startNextCycle(newSession.sessionId)"
      >Новый цикл</b-button
    >
    <p v-if="cycleRes.sensors !== null">{{ cycleRes }}</p>
  </section>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  data: () => ({
    newSession: {},
    nextCycle: {
      MoveUpFast: false,
      MoveUpSlow: false,
      MoveDownFast: false,
      MoveDownSlow: false,
      DoClose: false,
      DoOpen: false,
    },
    cycleRes: {
      sensors: null,
      emulation: { Alarm: null },
    },
    options: [{ text: 'Выбери одно', value: null }, true, false],
    show: true,
  }),

  async mounted() {
    this.newSession = await this.$axios.$post(
      'https://lstu.fusionsoft.ru/srv-elevator/newSession?timeFactor=1.0&floors=3&clientName=захар'
    )
  },

  methods: {
    async startNextCycle(sessionId: number) {
      this.cycleRes = await this.$axios.$put(
        `https://lstu.fusionsoft.ru/srv-elevator/nextCycle?sessionId=${sessionId}`,
        this.nextCycle
      )
      if (await this.cycleRes.emulation.Alarm) {
        alert(this.cycleRes.emulation.Alarm)
      }
    },
    onSubmit(event: any) {
      const res = confirm('Отправляем?')
      console.log(res)
      if (res) {
        event.preventDefault()
      } else {
        this.onReset(event)
      }
    },
    onReset(event: any) {
      event.preventDefault()
      // Сбросить значения нашей формы
      this.nextCycle.MoveUpFast = false
      this.nextCycle.MoveUpSlow = false
      this.nextCycle.MoveDownSlow = false
      this.nextCycle.DoClose = false
      this.nextCycle.DoOpen = false
      // Уловка для сброса/очистки состояния проверки формы в собственном браузере
      this.show = false
      this.$nextTick(() => {
        this.show = true
      })
    },
  },
})
</script>

<style scoped>
section {
  text-align: center;
}

.form {
  margin: 0 auto;
  max-width: 20%;
}
</style>
