<script>
import * as mqtt from "mqtt/dist/mqtt.min";

export default {
  name: 'App',
  data() {
    return {
      msg: 'Vue & mqtt',
      connection: {
        host: 'broker.emqx.io',
        port: 8083,
        endpoint: '/mqtt',
        clean: true, // 保留会话
        connectTimeout: 4000, // 超时时间
        reconnectPeriod: 4000, // 重连时间间隔
        // 认证信息
        clientId: 'mqttjs_3be2c321',
        username: 'emqx_test',
        password: 'emqx_test',
      },
      subscription: {
        topic: '/cxy1/led',
        qos: 0,
      },
      publication: {
        topic: '/cxy1/led',
        qos: 0,
        payload: 'on'   //'{ "msg": "Hello, I am browser." }',
      },
      receiveNews: '',
      qosList: [
        { label: 0, value: 0 },
        { label: 1, value: 1 },
        { label: 2, value: 2 },
      ],
      client: {
        connected: false,
      },
      subscribeSuccess: false,
      connectionOk: false
    }
  },
  // created(){
  //   this.createConnection();
  //   this.doSubscribe();
  //   this.doPublish();
  //   // this.doUnSubscribe();
  //   // this.destroyConnection();
  // },
  methods: {
    // 创建连接
    createConnection() {
      // 连接字符串, 通过协议指定使用的连接方式
      // ws 未加密 WebSocket 连接
      // wss 加密 WebSocket 连接
      // mqtt 未加密 TCP 连接
      // mqtts 加密 TCP 连接
      // wxs 微信小程序连接
      // alis 支付宝小程序连接
      const { host, port, endpoint, ...options } = this.connection
      const connectUrl = `ws://${host}:${port}${endpoint}`
      try {
        this.client = mqtt.connect(connectUrl, options)
      } catch (error) {
        console.log('mqtt.connect error', error)
      }
      this.client.on('connect', () => {
        console.log('Connection succeeded!');
        this.connectionOk = true;
      })
      this.client.on('reconnect', () => {
        console.log('reconnect');
        this.connectionOk = true;
      })      
      this.client.on('error', error => {
        console.log('Connection failed', error);
        this.connectionOk =false;
      })
      this.client.on('message', (topic, message) => {
        this.receiveNews = this.receiveNews.concat(message)
        console.log(`Received message ${message} from topic ${topic}`)
      })
    },
    doSubscribe() {
      const { topic, qos } = this.subscription
      this.client.subscribe(topic, qos, (error, res) => {
        if (error) {
          console.log('Subscribe to topics error', error)
          return
        }
        this.subscribeSuccess = true
        console.log('Subscribe to topics res', res)
      })
    },
    doUnSubscribe() {
      const { topic } = this.subscription
      this.client.unsubscribe(topic, error => {
        if (error) {
          console.log('Unsubscribe error', error)
        }
      })
    },
    doPublish() {
      const { topic, qos, payload } = this.publication
      this.publication.payload = this.publication.payload === 'on' ? 'off' :'on';
      this.client.publish(topic, payload, qos, error => {
        if (error) {
          console.log('Publish error', error)
        }
      })
    },
    destroyConnection() {
      if (this.connectionOk) {
        try {
          this.client.end()
          this.client = {
            connected: false,
          }
          this.connectionOk=false;
          console.log('Successfully disconnected!')
        } catch (error) {
          console.log('Disconnect failed', error.toString())
        }
      }
    }
  }
}
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <h1>hello {{ msg }}</h1>
      <button @click="createConnection" :disabled="connectionOk">连接MQTT服务器</button>
      <button @click="destroyConnection" :disabled="!connectionOk">断开MQTT服务器</button>
      <button @click="doSubscribe" :disabled="!connectionOk">订阅主题led</button>
      <button @click="doUnSubscribe" :disabled="!connectionOk">取消订阅led</button>
      <button @click="{ doPublish();} " :disabled="!connectionOk">开关led</button>
    </div>
  </header>

  <main>

  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
