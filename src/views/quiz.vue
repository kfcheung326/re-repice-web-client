<template>
  <div class="container">
    <h2>Matach Match Game</h2>
    <div class="row">
      <div class="d-flex flex-row justify-content-center py-3">
        <div class="turns p-3">
          <span class="btn btn-info">
            Turns :
            <span class="badge" :class="finish ? 'badge-success' : 'badge-light'">{{turns}}</span>
          </span>
        </div>
        <div class="totalTime p-3">
          <span class="btn btn-info">
            Total Time :
            <span
              class="badge"
              :class="finish ? 'badge-success' : 'badge-light'"
            >{{min}} : {{sec}}</span>
          </span>
        </div>
        <div class="totalTime p-3">
          <button class="btn btn-info" @click="reset" :disabled="!start">Restart</button>
        </div>
      </div>
      <div class="col-md-6 col-lg-6 col-xl-5 mx-auto">
        <div class="row justify-content-md-center">
          <div
            v-for="card in memoryCards"
            :key="card.id"
            class="col-auto mb-3 flip-container"
            :class="{ 'flipped': card.isFlipped, 'matched' : card.isMatched  }"
            @click="flipCard(card)"
          >
            <div class="memorycard">
              <div class="front border rounded shadow">
                <img width="100" height="150" src="../image/wp.jpg" />
              </div>
              <div class="back rounded border">
                <img width="100" height="150" :src="card.img" />
                <audio>
                  <source :src="card.img" type="audio/mpeg" />
                </audio>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<style scoped src="../style/game.css">
</style>

<script>
import _ from 'lodash';
export default {
  name: 'quiz',
  data() {
    return {
      memoryCards: [],
      start: false,
      turns: 0,
      totalTime: {
        minutes: 0,
        seconds: 0,
      },
      flippedCards: [],
      finish: false,
      cards: [
        {
          name: 'cheetah',
          img: [require('../image/wCheetah.jpg')],
          mp3: [require('../mp3/cheetah.mp3')],
        },
        {
          name: 'panda',
          img: [require('../image/wPanda.jpg')],
          mp3: [require('../mp3/panda.mp3')],
          
        },
        {
          name: 'alligator',
          img: [require('../image/wAlligator.jpg')],
          mp3: [require('../mp3/alligator.mp3')],

        },
        {
          name: 'zebra',
          img: [require('../image/wZebra.jpg')],
          mp3: [require('../mp3/zebra.mp3')],

        },
        {
          name: 'polarBear',
          img: [require('../image/wPolarBear.jpg')],
          mp3: [require('../mp3/polarBear.mp3')],

        },
        {
          name: 'kangaroo',
          img: [require('../image/wkangaroo.jpg')],
          mp3: [require('../mp3/kangaroo.mp3')],

        },
      ],
    };
  },
  methods: {
    count() {
      return (this.mark += 2);
    },

    flipCard(card) {
      if (card.isMatched || card.isFlipped || this.flippedCards.length === 2)
        return;

      if (!this.start) {
        this._startGame();
      }
      new Audio(card.mp3).play();
      card.isFlipped = true;
      if (this.flippedCards.length < 2) this.flippedCards.push(card);
      if (this.flippedCards.length === 2) this._match(card);
    },

    _startGame() {
      this._tick();
      this.interval = setInterval(this._tick, 1000);
      this.start = true;
    },

    _tick() {
      if (this.totalTime.seconds !== 59) {
        this.totalTime.seconds++;
        return;
      }

      this.totalTime.minutes++;
      this.totalTime.seconds = 0;
    },

    _match(card) {
      this.turns++;
      if (this.flippedCards[0].name === this.flippedCards[1].name) {
        setTimeout(() => {
          this.flippedCards.forEach((card) => (card.isMatched = true));
          this.flippedCards = [];
          if (this.memoryCards.every((card) => card.isMatched === true)) {
            clearInterval(this.interval);
            this.finish = true;
          }
        }, 400);
      } else {
        setTimeout(() => {
          this.flippedCards.forEach((card) => {
            card.isFlipped = false;
          });
          this.flippedCards = [];
        }, 800);
      }
    },
    reset() {
      clearInterval(this.interval);

      this.cards.forEach((card) => {
        card.isFlipped = false;
        card.isMatched = false;
      });

      setTimeout(() => {
        this.memoryCards = [];
        this.memoryCards = _.shuffle(
          this.memoryCards.concat(
            _.cloneDeep(this.cards),
            _.cloneDeep(this.cards)
          )
        );
        this.totalTime.minutes = 0;
        this.totalTime.seconds = 0;
        this.start = false;
        this.finish = false;
        this.turns = 0;
        this.flippedCards = [];
      }, 600);
    },
  },
  created() {
    this.reset();
    // this.cards.forEach((card) => {
    //   card.isFlipped = false;
    //   card.isMatched = false;
    // });

    // this.memoryCards = _.shuffle(
    //   this.memoryCards.concat(_.cloneDeep(this.cards), _.cloneDeep(this.cards))
    // );
  },
  computed: {
    sec() {
      if (this.totalTime.seconds < 10) {
        return '0' + this.totalTime.seconds;
      }
      return this.totalTime.seconds;
    },
    min() {
      if (this.totalTime.minutes < 10) {
        return '0' + this.totalTime.minutes;
      }
      return this.totalTime.minutes;
    },
    currentUser() {
      return this.$store.state.auth.user;
    },
  },
  mounted() {
    if (!this.currentUser) {
      this.$router.push('/login');
    }
  },
};
</script>
