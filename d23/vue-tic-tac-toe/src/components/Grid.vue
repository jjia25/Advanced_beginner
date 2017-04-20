<template>
  <div class="gameStatus" :class="gameStatusColor">
      {{ gameStatusMessage }}
  </div>
  <table class="grid">
    <tr>
      <cell name="1"></cell>
      <cell name="2"></cell>
      <cell name="3"></cell>
    </tr>
    <tr>
      <cell name="4"></cell>
      <cell name="5"></cell>
      <cell name="6"></cell>
    </tr>
    <tr>
      <cell name="7"></cell>
      <cell name="8"></cell>
      <cell name="9"></cell>
    </tr>
  </table>
</template>

<script>
import Cell from './Cell.vue'

export default {
  components: { Cell },
  data () {
    return {
          // can be O or X
     activePlayer: 'O',
     // maintains the status of the game: turn or win or draw
     gameStatus: 'turn',

     gameStatusMessage: `O's turn`,
     // status color is used as background color in the status bar
     // it can hold the name of either of the following CSS classes
     // statusTurn (default) is yellow for a turn
       // statusWin is green for a win
       // statusDraw is purple for a draw
     gameStatusColor: 'statusTurn',
     // no. of moves played by both players in a single game (max = 9)
     moves: 0,
     // stores the placement of X and O in cells by their cell number
       cells: {
           1: '', 2: '', 3: '',
           4: '', 5: '', 6: '',
           7: '', 8: '', 9: ''
       },
       // contains all (8) possible winning conditions
       winConditions: [
           [1, 2, 3], [4, 5, 6], [7, 8, 9], // rows
           [1, 4, 7], [2, 5, 8],    [3, 6, 9], // columns
           [1, 5, 9], [3, 5, 7]             // diagonals
       ],
    }
  },
  computed: {
    //gets the non-active player
    nonActivePlayer () {
      if(this.activePlayer === '0') {
        return 'X'
      }
      return 'O'
    }
  },
  watch: {
    gameStatus () {
      if (this.gameStatus === 'win') {
        this.gameStatusColor = 'statusWin'
        this.gameStatusMessage = '${this.activePlayer} Wins !'
        return
      } else if (this.gameStatus === 'draw') {
        this.gameStatusColor = 'statusDraw'
        this.gameStatusMessage = 'Draw !'
        return
      }
      this.gameStatusMessage = '${this.activePlayer}'s turn'
    }
  }
  methods:  {
    changePlayer () {
      this.activePlayer = this.nonActivePlayer
    }
    strike () {
      if (! this.frozen) {
        //gets X or O from Grid component
        this.mark = this.$parent.active.activePlayer
        this.frozen = true
        //fires event to notify Grid component a mark is placed
        Event.$emit('strike', this.name)
      }
    },
    created () {
      //listens for a strike made by the user on cell
      //called by Cell component
      Event.$on('strike', (cellNumber) =>{
        //sets X or O in clicked cell of cell array
        this.cells[cellNumber] = this.activePlayer
        //increments number of moves
        this.moves++
        //stores game status by calling changeGameStatus method
        this.gameStatus = this.changeGameStatus()
        this.changePlayer()
      }),
      Event.$on('freeze', () =>this.frozen = true),
      Event.$on('win', winner => this.wins[winner]++),
      Event.$on('gridReset', () => {
        Object.assign(this.$data, this.$options.data())
      })
    },
    //returns the game status to gameStatus property
    changeGameStatus () {
      if (this.checkForWin()) {
        return this.gameIsWon()
      //checks to see if game is not won and all cells are filled
      } else if (this.moves === 9) {
        return 'draw'
      }
      //sets status to turn
      return 'turn'
    },
    checkForWin () {
      for (let i = 0; i < this.winConditions.length; i++) {
        //gets a single condition wc from the whole array
        let wc = this.winConditions[i]
        let cells = this.cells

        //compares 3 cell values based on the cells in the condition
        if (this.areEqual(cells[wc[0]], cells[wc[1]], cells[wc[2]])) {
          return true
        }
      }
      return false
    },
    //compares cell values
    areEqual () {
      var len = arguments.length;
      //loops through each value and compares them with an empty string
      //& inequality
      for (var i = 1; i < len; i++) {
        if (arguments[i] === '' || arguments[i] !== arguments[i - 1])
          return false;
      }
      return true;
    },
    gameIsWon () {
      //fires event for App component to change score
      Event.$emit('win', this.activePlayer)
      //sets game status message
      this.gameStatusMessage = '${this.activePlayer} Wins !'
      //fires event for cell to freeze
      Event.$emit('freeze')
      //sets status to win
      return 'win'
    }
  }
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
