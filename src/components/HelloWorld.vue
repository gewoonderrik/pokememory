<template>
  <div class="hello">
    <div class="player player-one" :class="'player-one-active-' + playerOne.active">
    </div>

    <div class="player player-two" :class="'player-two-active-' + playerTwo.active">
    </div>

    <div class="deck">
      <div class="debug" style="display: none;">
        <pre>
          game started: {{ gameRunning }}<br>
          number of cards: {{ numberCards }}<br>
          playerOne: {{ playerOne.active }} {{ playerOne.score }}<br>
          playerTwo: {{ playerTwo.active }} {{ playerTwo.score }}<br>
          <br>
          {{ pokedeck }}
        </pre>
      </div>

      <div class="controls">
        <div class="controls-inactive" v-if="gameRunning === false">
          <input type="number" min="6" max="18" v-model="numberCards" />

          <button class="deck-button" @click="startGame()">
            Start
          </button>
        </div>

        <div class="controls-inactive" v-if="gameRunning === true">
          <button class="deck-button" @click="stopGame()">
            Stop
          </button>
        </div>
      </div>

      <div class="deck-cards">
        <div class="deck-cards-card" :class="'deck-cards-card-clicked-' + pokemon.clicked + ' deck-cards-card-guessed-' + pokemon.guessed + ' deck-cards-card-guessedby-' + pokemon.guessedBy" :data-index="index" v-bind:key="index" v-for="(pokemon, index) in pokedeck" @click="memoryCheck(index)">
          <div class="deck-cards-card-content">
            <img :src="pokemon.image" />
          </div>

          <div class="deck-cards-card-question">
            ?
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import Vue from 'vue';
import VueAxios from 'vue-axios';

Vue.use(VueAxios, axios);

const points = 1000;
const sec = 1000;

export default {
  name: 'HelloWorld',

  data: function() {
    return {
      gameRunning: false,
      numberCards: 6,
      offset: 20,
      pokedeck: [],
      guessedCards: 0,
      secondClick: null,
      secondClickId: null,
      firstClick: null,
      firstClickId: null,
      playerOne: {
        active: true,
        score: 0,
      },
      playerTwo: {
        active: false,
        score: 0,
      }
    }
  },

  methods: {
    /**
     * stopGame
     */
    stopGame() {
      this.gameRunning = false;
      this.numberCards = 6;
      this.offset = 20;
      this.pokedeck = [];
    },

    /**
     * memoryCheck
     */
    memoryCheck(clickedIndex) {
      this.setClicked(clickedIndex);
    },

    /**
     * Set clicked
     */
    setClicked(index) {
      this.pokedeck[index].clicked = true;

      if(this.firstClick === null) {
        this.firstClickId = index;
        this.firstClick = this.pokedeck[index];
      } else {
        this.secondClickId = index;
        this.secondClick = this.pokedeck[index];

        if(this.firstClick.name == this.secondClick.name) {
          if(this.firstClickId !== this.secondClickId) {
            this.guessedCards = this.guessedCards + 1;

            this.firstClick.guessed = true;
            this.secondClick.guessed = true;

            this.firstClick.guessedBy = this.getActivePlayer();
            this.secondClick.guessedBy = this.getActivePlayer();

            if(this.playerOne.active === true) {
              this.playerOne.score = this.playerOne.score + points;
            } else {
              this.playerTwo.score = this.playerTwo.score + points;
            }

            if(this.guessedCards === (this.pokedeck.length / 2)) {              
              this.gameWon();
              this.resetCount();
            }
          }
        } else {
          this.pokedeck.forEach((element) => {
            if(element.guessed == false) {
              setTimeout(() => element.clicked = false, sec);
            }
          });

          if(this.playerOne.active === true) {
            this.playerOne.active = false;
            this.playerTwo.active = true;
          } else {
            this.playerOne.active = true;
            this.playerTwo.active = false;             
          }
        }

        this.resetCount();
      }
    },

    /**
     * getActivePlayer
     */
    getActivePlayer() {
      if(this.playerOne.active) {
        return 'one';
      } else {
        return 'two';
      }
    },

    /**
     * gameWon
     */
    gameWon() {
      console.log('game won!');
    },

    /**
     * resetCount
     */
    resetCount() {
      this.firstClick = null;
      this.firstClickId = null;
      this.secondClick = null;
      this.secondClickId = null;
    },

    /**
     * startGame
     */
    startGame() {
      this.getDeck();
      this.gameRunning = true;
    },

    /**
     * shuffle
     */
    shuffle(array) {
      let currentIndex = array.length, temporaryValue, randomIndex;

      while (0 !== currentIndex) {
        randomIndex = Math.floor(Math.random() * currentIndex);
        currentIndex -= 1;

        temporaryValue = array[currentIndex];
        array[currentIndex] = array[randomIndex];
        array[randomIndex] = temporaryValue;
      }

      return array;
    },

    /**
     * getDeck
     */
    getDeck() {
      let url = 'https://pokeapi.co/api/v2/pokemon/?offset=' + this.offset + '&limit=' + this.numberCards;

      Vue.axios.get(url).then((response) => {
        response.data.results.forEach((element, index) => {
          Vue.axios.get(element.url).then((response) => {
            console.log(response.data);
    
            let firstEntry = {
              'clicked': false,
              'guessed': false,
              'guessedBy': '',
              'image': response.data.sprites.front_default,
              'name': response.data.name,
              'ogIndex': index,
              'type': response.data.type,
            }

            let secondEntry = {
              'clicked': false,
              'guessed': false,
              'guessedBy': '',
              'image': response.data.sprites.front_default,
              'name': response.data.name,
              'ogIndex': index,
              'type': response.data.type,
            }

            this.pokedeck.push(firstEntry);
            this.pokedeck.push(secondEntry);

            this.shuffle(this.pokedeck);
          });
        });
      });
    }
  }
}
</script>

<style scoped>
  .debug {
    float: left;
    font-family: monospace;
    font-size: 12px;
    max-width: 420px;
    text-align: left;
    width: 100%;
  }
  
  .hello {
    height: 100vh;
    position: relative;
    width: 100%;
  }

  .player {
    background: #192E8D;
    bottom: 0;
    height: 100vh;
    position: absolute;
    top: 0;
    width: 20vw;
    transition: all 0.5s 1s ease-out;
  }

    .player-one {
      box-shadow: 3px 0 3px rgba(0, 0, 0, 0.125);
      left: -20vw;
      opacity: 0;
    }

      .player-one-active-true {
        left: 0;
        opacity: 1;
      }

    .player-two {
      box-shadow: -3px 0 3px rgba(0, 0, 0, 0.125);
      opacity: 0;
      right: -20vw;
    }

      .player-two-active-true {
        opacity: 1;
        right: 0;
      }

  .deck {
    float: left;
    margin: 0 20vw;
    width: 60vw;
  }

    .deck-cards {
      float: left;
      width: 100%;
    }

      .deck-cards-card {
        box-shadow: 0 0 3px rgba(0, 0, 0, 0.25);
        background-color: #E8EAF0;
        float: left;
        height: 200px;
        margin: 20px;
        position: relative;
        transform: rotateY(-180deg);
        transition: all 0.25s ease-in;
        width: 150px
      }

        .deck-cards-card img {
          left: 50%;
          opacity: 0;
          position: absolute;
          top: 50%;
          transform: translateY(-50%) translateX(-50%);
          transition: all 0.25s ease-in;
        }

        .deck-cards-card-question {
          background-color: #192E8D;
          border-radius: 100%;
          color: #f4b743;
          font-family: serif;
          font-size: 30px;
          font-weight: 700;
          height: 40px;
          left: 50%;
          line-height: 40px;
          position: absolute;
          text-align: center;
          top: 50%;
          transform: translateX(-50%) translateY(-50%)rotateY(180deg);
          transition: all 0.25s ease-in;
          width: 40px;
        }

        .deck-cards-card-clicked-true {
          background-color: #fff;
          pointer-events: none;
          transform: scale(1.1) rotateY(0);
        }

          .deck-cards-card-clicked-true img {
            opacity: 1;
          }
        
          .deck-cards-card-clicked-true .deck-cards-card-question {
            opacity: 0;
            transform: scale(0);
          }
        
        .deck-cards-card-guessed-true {
          pointer-events: none;
        }

        .deck-cards-card-guessedby-one {
          background-color: red;
        }

        .deck-cards-card-guessedby-two {
          background-color: green;
        }
</style>
