<template>
  <div>
    <div class="gameStatus" :class="gameStatusColor">
      {{ gameStatusMessage }}
    </div>

    <table class="grid">
      <tr>
        <Cell name="1"></Cell>
        <Cell name="2"></Cell>
        <Cell name="3"></Cell>
      </tr>
      <tr>
        <Cell name="4"></Cell>
        <Cell name="5"></Cell>
        <Cell name="6"></Cell>
      </tr>
      <tr>
        <Cell name="7"></Cell>
        <Cell name="8"></Cell>
        <Cell name="9"></Cell>
      </tr>
    </table>
  </div>
</template>

<script>
  import Cell from './Cell.vue'
  export default {
    data () {
      return {
        activePlayer: 'O',
        // turn, win, or draw
        gameStatus: 'turn',
        gameStatusMessage: `O's turn`,
        // can be statusTurn (default), statusWin (green), or statusDraw (purple)
        gameStatusColor: 'statusTurn',
        // max 9
        moves: 0,
        cells: {
          1: '', 2: '', 3: '', 
          4: '', 5: '', 6: '', 
          7: '', 8: '', 9: ''
        }, 
        winConditions: [
          [1, 2, 3], [4, 5, 6], [7, 8, 9],  // rows
          [1, 4, 7], [2, 5, 8], [3, 6, 9],  // columns
          [1, 5, 9], [3, 5, 7]              // diagonals
        ]


      }
    },

    created () {
      Event.$on('strike', (cellNumber) => {
        this.cells[cellNumber] = this.activePlayer
        this.moves++
        this.gameStatus = this.changeGameStatus()
        this.changePlayer()
        this.gameStatusMessage = `${this.activePlayer}'s turn`
      })

      Event.$on('gridReset', () => {
        Object.assign(this.$data, this.$options.data())
      })
    },

    methods: {
      changePlayer () {
        this.activePlayer = this.nonActivePlayer
      },

      changeGameStatus () {
        if(this.checkForWin()) {
          return this.gameIsWon()
        } else if (this.moves === 9) {
          return 'draw'
        }
        return 'turn'
      },
      
      checkForWin () {
        for (let i = 0; i < this.winConditions.length; i++) {
          let wc = this.winConditions[i]
          let cells = this.cells

          if (this.areEqual(cells[wc[0]], cells[wc[1]], cells[wc[2]])) {
            return true
          }
        }

        return false
      },

      areEqual () {
        let len = arguments.length
        for (let i = 1; i < len; i++) {
          if (arguments[i] === '' || arguments[i] !== arguments[i-1]) {
            return false
          }
        }
        return true
      },

      gameIsWon () {
        Event.$emit('win', this.activePlayer)

        Event.$emit('freeze')

        return 'win'
      }


    },

    watch: {
      gameStatus () {
        if (this.gameStatus === 'win') {
          this.gameStatusColor = 'statusWin'
          this.gameStatusMessage = `${this.nonActivePlayer} Wins!`
          return
        } else if (this.gameStatus === 'draw') {
          this.gameStatusColor = 'statusDraw'
          this.gameStatusMessage = 'Draw!'
          return
        }
        this.gameStatusMessage = `${this.activePlayer}'s turn`
        console.log(this.activePlayer)
      }
    },

    computed: {
      nonActivePlayer () {
        if (this.activePlayer === 'O') {
          return 'X'
        }
        return 'O'
      }
    },

    components: {
      Cell
    },
  }
</script>

<style>
  .grid {
    background-color: #34495e;
    color: #fff;
    width: 100%;
    border-collapse: collapse;
  }

  .gameStatus {
    margin: 0px;
    padding: 15px;
    border-top-left-radius: 20px;
    border-top-right-radius: 20px;
    background-color: #f1c40f;
    color: #fff;    
    font-size: 1.4em;
    font-weight: bold;
  }

  .statusTurn {
      background-color: #f1c40f;
  }

  .statusWin {
      background-color: #2ecc71;
  }

  .statusDraw {
      background-color: #9b59b6;
  }
</style>