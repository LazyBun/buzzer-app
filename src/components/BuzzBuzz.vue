<template>
  <q-layout ref="layout" view="lHh Lpr fff" :left-class="{'bg-faded': true}">
    <q-toolbar slot="header" color="dark">
      <q-btn flat @click="$refs.layout.toggleLeft()">
        <q-icon name="menu" />
      </q-btn>

      <q-toolbar-title>
        Buzz Buzz
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
import { QLayout, QToolbar, QToolbarTitle, QBtn, QIcon, QList, QListHeader, QItem, QItemSide, QItemMain, QInput } from 'quasar'

// TODO: change to import
let Chance = require('chance')
let chance = new Chance()

export default {
  name: 'BuzzBuzz',
  components: { QLayout, QToolbar, QToolbarTitle, QBtn, QIcon, QList, QListHeader, QItem, QItemSide, QItemMain, QInput },
  data: () => {
    return {
      // TODO: save in cookie?
      wsAddr: 'ws://',
      name: '',
      bigButton: { width: '50px', height: '50px' },
      socket: null
    }
  },
  methods: {
    sendToWs () {
      if (this.socket) {
        this.socket.send(this.name)
      } else {
         // TODO: Missing ws ip indication
        console.log('set ws!')
      }
    },
    setBigButton (event) {
      let num = document.documentElement.clientWidth < document.documentElement.clientHeight ? document.documentElement.clientWidth / 2
        : document.documentElement.clientHeight / 2
      this.bigButton = { width: num + 'px', height: num + 'px' }
    },
    setWs() {
      if (this.wsAddr.length > 5) {
        // TODO: If error then indicate. Spinner when loading
        this.socket = new WebSocket(this.wsAddr)
        // TODO: Add this in - atm it would trigger game win
        // this.socket.onopen = () => {
        //   this.socket.send("Connected: " + this.name)
        // }
        this.socket.onmessage = (event) => {
          // TODO: Works for now, will be shit for more complex return data
          if (event.data === 'true') {
            // TODO: Winning indication
            console.log('nice')
          } else {
            // TODO: Winning indication
            console.log('too bad')
          }
        }
      }
    }
  },
  beforeDestroy () {
    window.addEventListener('resize', this.setBigButton)
  },
  mounted () {
    this.name = chance.name({ nationality: "it" })
    this.$nextTick(() => {
      window.addEventListener('resize', this.setBigButton)
      this.setBigButton(null)
    })
  }
}
</script>
