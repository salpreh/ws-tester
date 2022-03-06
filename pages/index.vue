<template>
  <b-container>
    <b-row >
      <b-card class="w-100 mb-4">
        <b-form @submit="sendToWs" @reset="resetForm">
          <b-form-group
            id="text-send-grp"
            label="WS message"
            label-for="text-send"
          >
            <b-form-input
              id="text-send"
              v-model="sendText"
              placeholder="data to send..."
            ></b-form-input>
          </b-form-group>

          <div class="mt-4">
            <b-button
              variant="outline-primary"
              @click="showUrlModal = true"
            >Set WS URL</b-button>
            <b-button
              variant="outline-success"
              @click="initWsConnection"
            >Reconnect</b-button>
            <b-button
              variant="outline-danger"
              @click="closeWsConnection"
            >Close connection</b-button>

            <b-button
              type="submit"
              variant="primary"
              class="float-right ml-1"
            >Submit</b-button>
            <b-button
              type="reset"
              variant="danger"
              class="float-right ml-1"
            >Clear</b-button>
          </div>
        </b-form>
      </b-card>
    </b-row>
    <b-row>
      <b-table
        :hover="true"
        :bordered="true"
        head-variant="dark"
        :items="eventsData"
      ></b-table>
    </b-row>

    <!-- Modals -->
    <b-modal
      v-model="showUrlModal"
      title="Set WS URL"
      ok-only
      @ok="initWsConnection"
    >
      <p>WebSocket URL</p>
      <b-form-input v-model="ws.url" type="url"></b-form-input>
    </b-modal>
  </b-container>
</template>

<script>
import moment from 'moment'
moment.defaultFormat = 'hh:mm:ss - DD/MM/YYYY'

export default {
  name: 'IndexPage',
  components: {},
  data: vm => ({
    tabFields: ['ts', 'data'],
    eventsData: [],
    ws: {
      url: "wss://missing",
      /** @type connection WebSocket **/
      connection: null
    },
    sendText: '',
    showUrlModal: false
  }),
  created() {
    this.initWsConnection()
  },
  mounted() {
    this.eventsData.push(this.addEvent(moment(), 'App init', 'info'))
  },
  methods: {
    sendToWs(ev) {
      ev.preventDefault()
      if (this.ws.connection == null) {
        alert('No connection stablished to WS')
        return
      }

      this.ws.connection.send(this.sendText)
    },

    resetForm(ev) {
      ev.preventDefault()
      this.sendText = ''
    },

    initWsConnection(force=false) {
      if (this.ws.connection && !force) return
      if (force) this.closeWsConnection()
      console.log("Creating ws connection")
      this.ws.connection = new WebSocket(this.ws.url)

      /** @type ev MessageEvent **/
      this.ws.connection.onmessage = ev => {
        console.log("incoming ws msg", ev)
        this.eventsData.push(this.addEvent(moment(), ev.data))
      }

      this.ws.connection.onopen = ev => {
        console.log("WS connection successfully established", ev)
        this.eventsData.push(this.addEvent(moment(), 'WS connection established', 'success'))
      }

      this.ws.connection.onclose = ev => {
        console.log("WS connection closed", ev)
        this.eventsData.push(this.addEvent(
          moment(),
          `WS closed (cod ${ev.code})${ev.reason ? ': '+ev.reason : ''}`, 'danger')
        )
      }
    },

    closeWsConnection() {
      if (this.ws.connection != null) {
        this.ws.connection.close()
        this.ws.connection = null
      }
    },

    /**
     * @param datetime {moment.Moment}
     * @param data {string}
     * @param style {string}
     */
    addEvent(datetime, data, style=null) {
      let msg = {ts: datetime.format(), data: data}
      if (style) msg._rowVariant = style

      return msg
    }
  }
}
</script>
