<template>
  <div>
    <div class="text-center">
      <v-row justify="center" align-content="center">
        <v-chip label class="ma-2" :color="timer_color" text-color="white">
        サイクル回数 : {{ parseInt(cycle_count/2) }}
        </v-chip>
      </v-row>
      <v-progress-circular
        :rotate="-90"
        :size="timer_size"
        :width="100"
        :value="value"
        :color="timer_color"
      >
        <h1>{{ formatted_timer }}</h1>
      </v-progress-circular>
    </div>
    <v-row class="justify-center mt-4">
      <vue-timepicker
        format="mm:ss"
        v-model="time_obj"
        @input="set_second"
        @open="timer_reset"
      ></vue-timepicker>
    </v-row>
    <v-row class="justify-center mt-3">
      <v-btn
        color="green"
        class="mt-2 white--text mx-2"
        @click="timer_start()"
        :disabled="!paused"
        >start</v-btn
      >
      <v-btn
        color="yellow"
        class="mt-2 mx-2"
        @click="timer_pause()"
        :disabled="paused"
        >pause</v-btn
      >
      <v-btn
        color="error"
        class="mt-2 mx-2"
        @click="timer_reset()"
        :disabled="!paused"
        >reset</v-btn
      >
    </v-row>
  </div>
</template>
<script>
import sound from '@/static/sounds/notificate.mp3'
import Push from 'push.js'
import VueTimepicker from 'vue2-timepicker/src/vue-timepicker.vue'
export default {
  components: {
    'vue-timepicker': VueTimepicker,
  },
  data() {
    return {
      cycle_count: 0,
      is_set_atom: false,
      interval: {},
      time_obj: {
        mm: '25',
        ss: '00',
      },
      timer_min: 0,
      timer_sec: 0,
      timer_second: 0,
      min: 0,
      sec: 0,
      second: 0,
      value: 100,
      paused: true,
      processing: true,
      atom: null,
    }
  },
  beforeDestroy() {
    clearInterval(this.interval)
  },
  computed: {
    timer_width() {
      switch (this.$vuetify.breakpoint.name) {
        case 'xs':
          return 90
        case 'sm':
          return 120
        case 'md':
          return 130
        case 'lg':
          return 150
        case 'xl':
          return 300
      }
      return 'hello'
    },
    timer_size() {
      switch (this.$vuetify.breakpoint.name) {
        case 'xs':
          return 300
        case 'sm':
          return 350
        case 'md':
          return 400
        case 'lg':
          return 500
        case 'xl':
          return 600
      }
      return 'hello'
    },
    timer_color() {
      return this.$store.state.timer_color
    },
    formatted_timer() {
      var min = Math.floor(this.second / 60)
      var sec = this.second % 60
      if (String(min).length == 1) {
        if (String(sec).length == 1) {
          return '0' + String(min) + ':0' + String(sec)
        } else {
          return '0' + String(min) + ':' + String(sec)
        }
      } else {
        if (String(sec).length == 1) {
          return String(min) + ':0' + String(sec)
        } else {
          return String(min) + ':' + String(sec)
        }
      }
    },
  },
  mounted() {
    if (Push.Permission.get() != 'granted') {
      Push.Permission.request()
    }
  },
  methods: {
    set_min(min) {
      if (String(min).length == 1) {
        this.time_obj['mm'] = '0' + String(min)
      } else {
        this.time_obj['mm'] = String(min)
      }
    },
    set_sec(sec) {
      if (String(sec).length == 1) {
        this.time_obj['ss'] = '0' + String(sec)
      } else {
        this.time_obj['ss'] = String(sec)
      }
    },
    set_atom() {
      this.set_second()
      this.atom = 100 / this.second
    },
    set_second() {
      console.log('set_second')
      this.second =
        Number(this.time_obj['mm']) * 60 + Number(this.time_obj['ss'])
    },
    timer_start() {
      if (!this.is_set_atom) {
        this.set_atom()
        this.is_set_atom = true
      }
      this.paused = false
      this.interval = setInterval(() => {
        this.second -= 1
        this.value -= this.atom
        // タイマーの時間を減らす
        this.set_min(Math.floor(this.second / 60))
        this.set_sec(this.second % 60)
        if (this.second < 0) {
          this.cycle_count++
          const audio = new Audio(sound)
          audio.play()
          Push.create('作業時間が終了しました！', {
            body: '5〜10分の休憩をしましょう！',
            onClick: function () {
              this.close()
            },
          })
          this.timer_reset()
          this.paused = true
        }
      }, 1000)
    },
    timer_pause() {
      this.set_min(Math.floor(this.second / 60))
      this.set_sec(this.second % 60)
      this.paused = true
      clearInterval(this.interval)
    },
    timer_reset() {
      this.is_set_atom = false
      clearInterval(this.interval)
      // サイクル回数が偶数回の場合
      if (this.cycle_count % 2 == 0) {
        this.time_obj['mm'] = '25'
        this.time_obj['ss'] = '00'
      } else {
        this.time_obj['mm'] = '05'
        this.time_obj['ss'] = '00'
      }
      this.second = 0
      this.value = 100
    },
  },
}
</script>
<style scoped>
.v-progress-circular {
  margin: 1rem;
}
</style>
