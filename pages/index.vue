<template>
  <section>
    <div>
      <b-navbar toggleable="sm" type="light" variant="light">
        <b-navbar-toggle target="nav-text-collapse"></b-navbar-toggle>

        <b-navbar-brand>{{ currentSession.sessionId }}</b-navbar-brand>

        <b-collapse id="nav-text-collapse" is-nav>
          <b-navbar-nav>
            <b-nav-text>{{ currentSession.closedSessionId }}</b-nav-text>
          </b-navbar-nav>
        </b-collapse>
      </b-navbar>
    </div>
    <main>
      <div class="house">
        <table width="20%" cellspacing="0" cellpadding="63">
          <tr v-for="floor in newSession.floors" :key="floor">
            <td width="200" valign="top"></td>
            <td class="lift" width="5" valign="top"></td>
            <td id="floor" valign="top">
              <button id="third_floor" inner-height="70"></button>
            </td>
          </tr>
        </table>
      </div>
      <div class="options">
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
          v-if="currentSession.sessionId"
          class="newCycle_button"
          @click.prevent="startNextCycle(currentSession.sessionId)"
          >Новый цикл</b-button
        >
      </div>
    </main>
    <p v-if="cycleRes.sensors !== null">{{ cycleRes }}</p>
  </section>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  data: () => ({
    newSession: {
      timeFactor: 1.0,
      floors: 5,
      clientName: 'захар',
    },
    currentSession: { sessionId: 0 },
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
    this.currentSession = await this.$axios.$post(
      'https://lstu.fusionsoft.ru/srv-elevator/newSession?timeFactor=1.0&floors=3&clientName=захар'
    )
    if (this.currentSession.sessionId) {
      setInterval(async () => {
        await this.startNextCycle(this.currentSession.sessionId)
      })
    }
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
.first_floor {
  font-size: 10;
}
.lift {
  background: url('../static/elevator-closed.svg'), no-repeat;
}
.house {
  margin: auto 0;
  background-color: whitesmoke;
}
section {
  text-align: center;
}

main {
  max-width: 40%;
  margin: 1% auto auto auto;
  display: flex;
  justify-content: space-between;
}

.options {
  margin: auto 0;
}

.form {
  margin-bottom: 2%;
}
</style>
