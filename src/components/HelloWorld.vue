<template>
  <div class="hello">
    <div class="item">
      <label for="light">灯光：</label>
      <div class="action">
        <p>
          <button @click="lightHanlder('on')">开灯</button>
          <button style="margin-left: 10px" @click="lightHanlder('off')">
            关灯
          </button>
        </p>
      </div>
    </div>
    <div class="item">
      <label for="light">温湿度：</label>
      <div class="action">
        <p>温度: <span>{{temp}}</span> ℃</p>
        <p style="margin-left: 40px">湿度: <span>{{humi}}</span> RH%</p>
      </div>
    </div>
  </div>
</template>

<script>
import * as ws from "websocket";
const WebSocketClient = ws.w3cwebsocket;
const type = {
  LIGHT_CTRL: 'LIGHT_CTRL',
  TEMP_HUMI: 'TEMP_HUMI'
}

export default {
  name: "HelloWorld",
  data() {
    return {
      cient: null,
      finished: false,
      temp: 0,
      humi: 0
    };
  },
  mounted() {
    const self = this
    const client = new WebSocketClient("ws://localhost:8888/", "jerry-demo");
    client.onerror = function () {
      console.log("Connection Error");
    };

    client.onopen = function () {
      console.log("WebSocket Client Connected");
    };

    client.onclose = function () {
      console.log("jerry-demo Client Closed");
    };

    client.onmessage = function (e) {
      if (typeof e.data === "string") {
        console.log("Received: '" + e.data + "'");
        self.parseMessage(JSON.parse(e.data))
      }
    };

    this.client = client;
  },
  methods: {
    sendToServer(payload) {
       this.client.send(JSON.stringify(payload))
    },
    lightHanlder(state) {
      console.log("do actoin: ", state);
      this.sendToServer({
        type: type.LIGHT_CTRL,
        payload: { state }
      })
    },
    parseMessage(msg) {
      if (msg.type === type.LIGHT_CTRL) {
        this.finished = true
      } else {
        this.temp = msg.payload.temp
        this.humi = msg.payload.humi
      }
    }
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
div.item {
  margin: 40px;
}
label {
  display: block;
  font-weight: bold;
  font-size: 24px;
  text-align: left;
}
div.action {
  display: flex;
}
div.action p span {
  font-size: 26px;
  font-weight: bold;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
