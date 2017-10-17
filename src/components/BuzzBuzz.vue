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
        <q-input v-model="wsAddr" float-label="Ws Address"/>
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
      bigButton: { width: '50px', height: '50px' }
    }
  },
  methods: {
    sendToWs () {
      if (this.wsAddr.length > 5) {
        // TODO: Initialize once when wsAddr is changed.
        let socket = new WebSocket(this.wsAddr)
        socket.onopen = () => {
          socket.send(this.name)
        }
        socket.onmessage = (event) => {
          // TODO: Works for now, will be shit for more complex return data
          if (event.data === 'true') {
            // TODO: Winning indication
            console.log('nice')
          } else {
            // TODO: Winning indication
            console.log('too bad')
          }
        }
      } else {
        // TODO: Missing ws ip indication
        console.log('set ws!')
      }
    },
    setBigButton (event) {
      let num = document.documentElement.clientWidth < document.documentElement.clientHeight ? document.documentElement.clientWidth / 2
        : document.documentElement.clientHeight / 2
      this.bigButton = { width: num + 'px', height: num + 'px' }
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
