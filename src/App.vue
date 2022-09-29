<template>
  <div>
    <header>
      <h1>FlipBOOM</h1>
    </header>
    <div id="mine_field">
      <div class="square" :class="{ flipped: square.flipped || lost || won, flagged: square.flagged, won: won && square.flagged, lost: lost && square.bomb }" v-for="(square, index) in squares" @click.left="flip(index)" @click.right.prevent="flag(index)">
        <div class="bomb" :class="{ exploded: square.exploded }" v-if="(square.flipped || lost) && square.bomb">
          💣
        </div>
        <div v-else-if="(square.flipped || lost)">
          {{ adjacentBombs(index) !== 0 ? adjacentBombs(index) : "&nbsp;" }}
        </div>
        <div class="flag" v-else-if="square.flagged">
          ⛳️
        </div>
        <div v-else>&nbsp;</div>
      </div>
    </div>
    <div class="won" v-if="won">You won!</div>
    <button class="reset" @click="initialize">Start Over</button>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'app',
  data() {
    return {
      squares: [
        {
          bomb: false,
          flagged: false,
          flipped: false,
          exploded: false
        }
      ],
      numBombs: 8,
      lost: false
    }
  },
  mounted() {
    this.initialize()
  },
  computed: {
    won() {
      let numFlagged = this.squares.filter(square => square.flagged).length
      // console.log(numFlagged);
      return numFlagged === this.numBombs
    }
  },
  methods: {
    initialize() {
      this.lost = false
      this.squares = []
      for (let i = 0; i < 64; i++) {
        this.squares.push({
          bomb: false,
          flagged: false,
          flipped: false,
          exploded: false,
        })
      }
      let bombs = 0;
      while (bombs < this.numBombs) {
        let ran = Math.floor(Math.random() * 64)
        if (!this.squares[ran].bomb) {
          this.squares[ran].bomb = true
          bombs++
        }
      }
    },
    flip(index: number) {
      if (this.squares[index].flipped || this.squares[index].flagged) {
        return false;
      }
      if (this.squares[index].bomb) {
        this.lost = true
        this.squares[index].exploded = true
      } else {
        this.squares[index].flipped = true
        if (this.adjacentBombs(index) == 0) {
          let adjacent = this.adjacent(index)
          adjacent.forEach((adjacentIndex) => {
            if (this.squares[adjacentIndex]) {
              if (!this.squares[adjacentIndex].bomb) {
                this.flip(adjacentIndex)
              }
            }
          })
        }
      }
      return
      console.log(this.won);
      
    },
    flag(index: number) {
      this.squares[index].flagged = !this.squares[index].flagged
    },
    adjacentBombs(index: number) {
      let adjacent = this.adjacent(index)
      let adjacentBombs = 0
      adjacent.forEach(adjacentIndex => {
        if (this.squares[adjacentIndex] && this.squares[adjacentIndex].bomb) {
          adjacentBombs++
        }
      })
      return adjacentBombs
    },
    adjacent(index: number) {
      if (index % 8 === 0) {
        return [index, index - 8, index - 7, index + 1, index + 8, index + 9]
      }
      if ((index - 7) % 8 === 0) {
        return [index, index - 1, index - 8, index - 9, index + 7, index + 8]
      }
      return [index, index - 1, index - 8, index - 9, index - 7, index + 1, index + 7, index + 8, index + 9]
    }
  }
})
</script>

<style scoped>
#mine_field {
  display: grid;
  grid-template-columns: repeat(8, 5em);
  grid-template-rows: repeat(8, 5em);
  grid-gap: 0;
  align-items: center;
  justify-items: center;
}
.square {
  display: grid;
  justify-items: center;
  align-items: center;
  font-size: 1.5em;
  margin: 0;
  padding: 5px;
  min-width: 5rem;
  min-height: 5rem;
  border: 1px solid lightgray;
  cursor: pointer;
  background-color: gray;
}
.flipped {
  background-color: white;
}
.flagged, .flipped {
  cursor: default;
}
.won {
  background-color: green;
}
.lost {
  background-color: red;
}
header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
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

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>
