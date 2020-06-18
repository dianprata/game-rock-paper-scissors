<template>
<div class="game">
  <h1>ROUND {{ round }}</h1>

  <b-row>
    <b-col md="6">
      <h3>YOUR SCORE</h3>
      <b-form-rating v-model="yourScore"
                     inline
                     stars="3"
                     size="lg"
                     readonly
                     class="mt-n3"
      ></b-form-rating>
      <div class="mt-3">
        <img src="@/assets/rock.svg"
             alt="rock"
             width="80px"
             class="cursor-pointer"
             v-show="yourChoose === 0 || yourChoose === 3"
             @click="yourChoose = 3">
        <img src="@/assets/paper.svg"
             alt="paper"
             width="80px"
             class="mx-5 cursor-pointer"
             v-show="yourChoose === 0 || yourChoose === 1"
             @click="yourChoose = 1">
        <img src="@/assets/scissors.svg"
             alt="scissors"
             width="80px"
             class="cursor-pointer"
             v-show="yourChoose === 0 || yourChoose === 2"
             @click="yourChoose = 2">
      </div>
    </b-col>
    <b-col md="6">
      <h3>OPPONENT'S SCORE</h3>
      <b-form-rating v-model="opponentsScore"
                     inline
                     stars="3"
                     size="lg"
                     readonly
                     class="mt-n3"
      ></b-form-rating>
      <div class="mt-3">
        <img src="@/assets/rock.svg"
             alt="rock"
             width="80px"
             class="cursor-pointer"
             v-show="opponentsChoose === 0 || opponentsChoose === 3"
             @click="opponentsChoose = 3">
        <img src="@/assets/paper.svg"
             alt="paper"
             width="80px"
             class="mx-5 cursor-pointer"
             v-show="opponentsChoose === 0 || opponentsChoose === 1"
             @click="opponentsChoose = 1">
        <img src="@/assets/scissors.svg"
             alt="scissors"
             width="80px"
             class="cursor-pointer"
             v-show="opponentsChoose === 0 || opponentsChoose === 2"
             @click="opponentsChoose = 2">
      </div>
    </b-col>
  </b-row>
  <h1>{{ result }}</h1>
  <router-link to="/" class="btn btn-outline-dark fixed-bottom">Back to Homepage</router-link>
</div>
</template>

<script>
import mqtt from 'mqtt';

export default {
  name: 'Game',
  data() {
    return {
      round: 1,
      yourScore: 0,
      yourChoose: 0,
      opponentsScore: 0,
      opponentsChoose: 0,
      result: '',
      rooms: [],
      room: {
        topic: '',
        currentPlayer: 0,
        players: [],
      },
      player: {
        clientId: '',
      },
    };
  },
  watch: {
    yourScore() {
      setTimeout(() => {
        if (this.yourScore === 3) {
          alert('YOU WIN');
          this.yourScore = 0;
          this.opponentsScore = 0;
          this.round = 1;
          this.result = '';
          this.reset();
        }
      }, 1000);
    },
    opponentsScore() {
      setTimeout(() => {
        if (this.opponentsScore === 3) {
          alert('YOU LOSE');
          this.yourScore = 0;
          this.opponentsScore = 0;
          this.round = 1;
          this.result = '';
          this.reset();
        }
      }, 1000);
    },
    yourChoose() {
      this.logicGame();
    },
    opponentsChoose() {
      this.logicGame();
    },
  },
  methods: {
    logicGame() {
      // Rock 3 > Scissors 2 > Paper 1 > Rock 3.
      if (this.yourChoose !== 0 && this.opponentsChoose !== 0) {
        if (this.yourChoose > this.opponentsChoose) {
          if (this.yourChoose === 3 && this.opponentsChoose === 1) {
            this.opponentsScore += 1;
            this.result = 'YOU LOSE';
          } else {
            this.yourScore += 1;
            this.result = 'YOU WIN';
          }
        }
        if (this.yourChoose < this.opponentsChoose) {
          if (this.yourChoose === 1 && this.opponentsChoose === 3) {
            this.yourScore += 1;
            this.result = 'YOU WIN';
          } else {
            this.opponentsScore += 1;
            this.result = 'YOU LOSE';
          }
        }
        if (this.yourChoose === this.opponentsChoose) {
          this.result = 'TIE';
        }
        this.round += 1;
        this.reset();
      }
    },
    reset() {
      this.yourChoose = 0;
      this.opponentsChoose = 0;
    },
    async connectMqtt() {
      let messageJson = {};
      const client = await mqtt.connect('ws://broker.mqttdashboard.com:8000/mqtt');
      client.on('connect', () => {
        client.subscribe('queue', (err) => {
          if (!err) {
            client.publish('queue', JSON.stringify({ topic: `room-${Math.random().toString(16).substr(2, 8)}`, player: 1 }));
          }
        });
      });
      client.on('message', (topic, message) => {
        const timer = setTimeout(() => {
          messageJson = { topic: Math.random(), player: 1 };
          client.publish(JSON.stringify(messageJson));
          client.unsubscribe('queue');
        }, 10 * 1000);
        messageJson = JSON.parse(message);
        if (messageJson.player === 1) {
          clearTimeout(timer);
          client.unsubscribe('queue');
        }
        this.connectPlayer(messageJson, client.clientId);
      });
    },
    connectPlayer(messages, clientId) {
      const client = mqtt.connect('ws://broker.mqttdashboard.com:8000/mqtt', { clientId });
      client.on('connect', () => {
        client.subscribe(messages.topic, (err) => {
          if (!err) {
            client.publish(messages.topic, clientId);
            console.log(messages.topic);
          }
        });
      });
    },
  },
  created() {
    this.connectMqtt();
  },
};
</script>

<style scoped>

</style>
