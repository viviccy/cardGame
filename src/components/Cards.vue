<template>
  <div>
    <h1>Card Game</h1>
    <p>
      Each player draw a card from the deck to determine which one is bigger.
      Player with the biggest number score one point. Who get the most points
      once all cards drawn is the winner.
    </p>
    <div class="cardHolder">
      <div class="player1Card">
        <component :is="p1Card" />
      </div>
      <div @click="drawCard">
        <div class="deck"><card /></div>
      </div>
      <div class="player2Card">
        <component :is="p2Card" />
      </div>
      <div>
        <p class="scoreText">P1 Score: {{ p1score }}</p>
      </div>
      <div>
        <div v-if="roundWinner">
          <div class="winnerText">Round Winner : {{ roundWinner }}</div>
          <br />
          <div>Next round in: {{ curTimer }} sec</div>
        </div>
        <div v-if="gameWinner">
          <div class="winnerText">Player {{ gameWinner }} is the champion!</div>
        </div>
      </div>
      <div>
        <p class="scoreText">P2 Score: {{ p2score }}</p>
      </div>
    </div>

    <button v-if="!start" @click="startGame">START GAME</button>
    <p v-if="startRound">Player {{ whoTurn }} GO!</p>
  </div>
</template>

<script>
//import card from "../assets/clubs_ace.svg"

let card = require("../assets/clubs_ace.svg")

export default {
  name: "Cards",
  components: {
    card,
  },
  data() {
    return {
      start: false,
      startRound: false,
      timer: 3,
      suitOrder: ["diamonds", "clubs", "hearts", "spades"],
      images: [],
      suits: [],
      player1Card: "card",
      player2Card: "card",
      player1Next: { cardName: "", cardNumber: "", suit: "" },
      player2Next: { cardName: "", cardNumber: "", suit: "" },
      p1score: 0,
      p2score: 0,
      whoTurn: 1,
      roundWinner: null,
      gameWinner: null,
      resetRound: false,
    }
  },
  mounted() {
    this.importAll(require.context("../assets/", true, /\.svg$/))
  },

  computed: {
    p1Card() {
      if (this.player1Next.cardName) {
        return () => import("../assets/" + this.player1Next.cardName)
      }
      return null
    },
    p2Card() {
      if (this.player2Next.cardName) {
        return () => import("../assets/" + this.player2Next.cardName)
      }
      return null
    },
    curTimer() {
      if (this.timer) {
        return this.timer
      }
      return null
    },
  },

  methods: {
    startGame() {
      this.start = true
      this.startRound = true
      this.roundWinner = null
      this.gameWinner = null
    },

    drawCard() {
      if (this.start && this.startRound) {
        let arrayIndex = Math.ceil(Math.random() * (this.images.length - 1))
        let pickedCard = this.images[arrayIndex].imageName
        let pickedCardVal = this.images[arrayIndex].value

        if (pickedCardVal == "jack") {
          pickedCardVal = 11
        } else if (pickedCardVal == "queen") {
          pickedCardVal = 12
        } else if (pickedCardVal == "king") {
          pickedCardVal = 13
        } else if (pickedCardVal == "ace") {
          pickedCardVal = 14
        }

        let pickedCardSuit = this.images[arrayIndex].suit

        if (this.whoTurn === 1) {
          this.player1Next.cardName = pickedCard
          this.player1Next.cardNumber = pickedCardVal
          this.player1Next.suit = pickedCardSuit
          this.whoTurn = 2
        } else {
          this.player2Next.cardName = pickedCard
          this.player2Next.cardNumber = pickedCardVal
          this.player2Next.suit = pickedCardSuit

          this.startRound = false

          this.whoWinRound()

          this.whoTurn = 1
        }

        if (arrayIndex > -1) {
          this.images.splice(arrayIndex, 1)
        }
      }
    },

    whoWinRound() {
      if (this.player1Next && this.player2Next) {
        if (
          Number(this.player1Next.cardNumber) >
          Number(this.player2Next.cardNumber)
        ) {
          this.roundWinner = "Player 1"
          this.p1score++
        } else if (
          Number(this.player1Next.cardNumber) <
          Number(this.player2Next.cardNumber)
        ) {
          this.roundWinner = "Player 2"
          this.p2score++
        } else if (
          Number(this.player1Next.cardNumber) ==
          Number(this.player2Next.cardNumber)
        ) {
          this.roundWinner = this.checkSuit(
            this.player1Next.suit,
            this.player2Next.suit
          )

          if (this.roundWinner == "Player 1") {
            this.p1score++
          } else {
            this.p2score++
          }
        }

        this.countDown(3)
      }
    },
    checkSuit(p1Suit, p2Suit) {
      let indexP1 = this.suitOrder.indexOf(p1Suit)
      let indexP2 = this.suitOrder.indexOf(p2Suit)

      if (indexP1 > indexP2) {
        return "Player 1"
      } else return "Player 2"
    },

    countDown(i) {
      let self = this
      let int = setInterval(function () {
        self.timer = i

        i-- || clearInterval(int)

        if (i == -1) {
          self.runNextRound()
          self.timer = 3
        }
      }, 1000)
    },

    runNextRound() {
      if (this.images.length > 1) {
        this.emptyObj(this.player1Next)
        this.emptyObj(this.player2Next)

        this.roundWinner = null
        this.startRound = true
      } else {
        this.announceWinner()
        this.roundWinner = null
        this.startRound = false
      }
    },
    announceWinner() {
      if (Number(this.p1score) > Number(this.p2score)) {
        this.gameWinner = 1
      } else if (Number(this.p1score) < Number(this.p2score)) {
        this.gameWinner = 2
      } else if (Number(this.p1score) == Number(this.p2score)) {
        this.gameWinner == "1 and 2"
      }

      this.start = false
      this.startRound = false
    },
    emptyObj(object) {
      Object.keys(object).forEach(function (k) {
        if (object[k] && typeof object[k] === "object") {
          return this.emptyObj(object[k])
        }
        object[k] = ""
      })
    },

    importAll(r) {
      r.keys().forEach((key) => {
        let currentCardName = key.replace("./", "")
        let currentCardValue = currentCardName.split(/[_.]+/)

        this.images.push({
          imageName: currentCardName,
          suit: currentCardValue[0],
          value: currentCardValue[1],
        })

        if (this.suits.indexOf(currentCardValue[0]) == -1) {
          this.suits.push(currentCardValue[0])
        }
      })
    },
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.cardHolder {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: 1fr auto;
  margin: 20px;

  .player1Card {
    /*   visibility: hidden; */
    position: relative;
  }

  .deck {
    display: inline-block;
    background-color: lightgreen;
    background-size: 50px 50px;
    background-image: linear-gradient(
        45deg,
        green 25%,
        transparent 25%,
        transparent
      ),
      linear-gradient(-45deg, green 25%, transparent 25%, transparent),
      linear-gradient(45deg, transparent 75%, green 75%),
      linear-gradient(-45deg, transparent 75%, green 75%);
    background-position-x: -7px;
    svg {
      visibility: hidden;
    }
  }

  .player2Card {
    /*  visibility: hidden; */
  }

  .winnerText {
    padding: 10px;
    font-weight: bold;
    font-size: 25pt;
  }
}
</style>
