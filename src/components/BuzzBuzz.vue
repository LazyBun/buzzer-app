<template>
  <q-layout ref="layout" view="lHh Lpr fff" :left-class="{'bg-faded': true}">
    <q-toolbar slot="header" color="dark">
      <q-btn flat @click="$refs.layout.toggleLeft()">
        <q-icon name="menu" />
      </q-btn>

      <q-toolbar-title>
        {{ name }} - Buzzer
      </q-toolbar-title>
    </q-toolbar>

    <div slot="left">
      <q-list no-border inset-delimiter>
        <div class="row justify-center items-center">
          <div class="col-8">
            <q-input v-model="wsAddr" float-label="Ws Address"/>
          </div>
          <div class="col-4">
            <q-btn style="width:100%" color="primary" @click="setWs"> Set </q-btn>
          </div>
        </div>
        <q-input v-model="name" float-label="Name"/>
      </q-list>
    </div>

    <div class="window-height window-width row justify-center items-center bg-dark">
      <div>
        <q-btn round big color="red" :style="bigButton" @click="sendToWs"/>  
      </div>
    </div>
  </q-layout>
</template>

<script>
import { QLayout, QToolbar, QToolbarTitle, QBtn, QIcon, QList, QListHeader, QItem, QItemSide, QItemMain, QInput, Toast, Cookies } from 'quasar'

// TODO: change to import
let Chance = require('chance')
let chance = new Chance()

let WS_COOKIE = 'buzzer_ws_cookie'
let NAME_COOKIE = 'buzzer_name_cookie'

export default {
  name: 'BuzzBuzz',
  components: { QLayout, QToolbar, QToolbarTitle, QBtn, QIcon, QList, QListHeader, QItem, QItemSide, QItemMain, QInput, Toast, Cookies },
  data: () => {
    return {
      wsAddr: 'ws://',
      name: '',
      bigButton: { width: '50px', height: '50px' },
      socket: null
    }
  },
  watch: {
    wsAddr: (val) => {
      Cookies.set(WS_COOKIE, val)
    },
    name: (val) => {
      Cookies.set(NAME_COOKIE, val)
    }
  },
  methods: {
    sendToWs () {
      if (this.socket) {
        switch(this.socket.readyState) {
          case this.socket.OPEN:
            this.socket.send("win:" + this.name); break;
          case this.socket.CONNECTING:
            Toast.create.warning({
              html: '<span style="color: black">Still connecting to websocket.</span>',
              timeout: 2000
            })
            break;
          case this.socket.CLOSING:
          case this.socket.CLOSED:
            Toast.create.negative({
              html: 'Connection either closing or closed. Reconnect by setting address again.',
              timeout: 2000
            })
            break;  
        }
      } else {
        Toast.create.negative({
          html: 'Set server address.',
          timeout: 5000
        })
      }
    },
    setBigButton (event) {
      let num = document.documentElement.clientWidth < document.documentElement.clientHeight ? document.documentElement.clientWidth / 2
        : document.documentElement.clientHeight / 2
      this.bigButton = { width: num + 'px', height: num + 'px' }
    },
    async setWs() {
      if (this.wsAddr.length > 5) {
        this.socket = await new WebSocket(this.wsAddr)
        this.socket.onopen = () => {
          this.socket.send("connect:" + this.name)
        }
        this.socket.onmessage = this.handleOnMessage
      }
    },
    handleOnMessage(event) {
      // TODO: Investigate why switch/case does not work (most likely something with == vs ===)
      // TODO: Better win/lose indication, for now - toast
      if (event.data === 'won') {
        Toast.create.positive({
          html: 'Winner winner, chicken dinner',
          timeout: 1000
        })
      } else if (event.data === 'lost') {
        Toast.create.negative({
          html: 'Bzzzzzzz',
          timeout: 500
        })
      } else if (event.data === 'connected') {
        Toast.create.positive({
          html: 'Connected!',
          timeout: 500
        })
      } else {
        Toast.create.negative({
          html: 'Unhandled or bad message sent. Please leave issue at <a href="https://github.com/LazyBun/buzzer-app">repository</a>',
          timeout: 3000
        })
      }
    }

  },
  beforeDestroy () {
    window.addEventListener('resize', this.setBigButton)
  },
  mounted () {
    if (Cookies.has(WS_COOKIE)) { this.wsAddr = Cookies.get(WS_COOKIE); this.setWs() }
    this.name = Cookies.has(NAME_COOKIE) ? Cookies.get(NAME_COOKIE) : chance.name({ nationality: "it" })

    this.$nextTick(() => {
      window.addEventListener('resize', this.setBigButton)
      this.setBigButton(null)
    })
  }
}
</script>
