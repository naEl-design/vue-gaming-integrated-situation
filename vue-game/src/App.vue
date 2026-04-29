<template>
  <div class="app">
    <h1>🧩 Rwandan Museum Puzzle Game</h1>
    <h2>Level {{ currentLevel + 1 }} / {{ historicalFigures.length }}</h2>
    <p class="score">Score: {{ score }}</p>

    <div class="board">
      <div
        v-for="(tile, index) in tiles"
        :key="index"
        class="tile"
        :class="{ empty: tile === null }"
        @click="moveTile(index)"
        :style="tile !== null ? getTileStyle(tile) : {}"
      >
        <span v-if="tile !== null">{{ tile }}</span>
      </div>
    </div>

    <div class="controls">
      <button @click="initializePuzzle">🔄 Try Again</button>
      <button @click="goToNextLevel">➡️ Next Level</button>
    </div>

    <p v-if="levelMessage" class="message">{{ levelMessage }}</p>

    <div class="levels-container">
      <h3>Upcoming Historical Figures</h3>
      <div class="levels-grid">
        <div
          v-for="(figure, index) in historicalFigures"
          :key="index"
          class="level-card"
          :class="{
            active: index === currentLevel,
            locked: index > unlockedLevel
          }"
        >
          <div v-if="index <= unlockedLevel">
            <img
              :src="`/src/assets/images/${figure.image}`"
              :alt="figure.name"
            />
            <p>{{ figure.name }}</p>
          </div>
          <div v-else>
            <div class="locked-box">🔒 Locked</div>
            <p>Level {{ index + 1 }}</p>
          </div>
        </div>
      </div>
    </div>

    <div v-if="showWinModal" class="modal">
      <div class="modal-content">
        <h2>🎉 You Win!</h2>
        <p>You earned {{ lastEarnedScore }} points!</p>
        <h3>{{ historicalFigures[currentLevel].name }}</h3>
        <p>{{ historicalFigures[currentLevel].fact }}</p>
        <button @click="closeWinModal">Continue</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      historicalFigures: [
   { name: 'kagame', image: 'kagame.jpg', fact: 'Rwabugiri expanded Rwanda and strengthened its leadership.' },
        { name: 'Rwigema', image: 'rwigema.jpg', fact: 'She was Rwanda’s first female Prime Minister.' },
        { name: 'King Mutara III Rudahigwa', image: 'hero3.jpg', fact: 'He helped modernize Rwanda and promoted unity.' },
        { name: 'Queen Mother Kanjogera', image: 'kanjogera.jpg', fact: 'A powerful queen mother in Rwanda’s royal history.' },
        { name: 'Ndabaga', image: 'ndabaga.jpg', fact: 'A legendary heroine known for bravery.' },
        { name: 'Yuhi V Musinga', image: 'yuhi musinga.jpg', fact: 'A former king during colonial transition.' },
        { name: 'Alexis Kagame', image: 'alexis kagame.jpg', fact: 'A major historian and scholar of Rwanda.' },
        { name: 'Michel Rwagasana', image: 'michel.jpg', fact: 'A political leader who advocated independence.' },
        { name: 'Félicité Niyitegeka', image: 'niyitegeka.jpg', fact: 'Recognized for courage and humanity.' },
      
      ],
      currentLevel: 0,
      unlockedLevel: 0,
      tiles: [],
      emptyIndex: 8,
      score: 0,
      moves: 0,
      showWinModal: false,
      lastEarnedScore: 0,
      completedCurrentLevel: false,
      levelMessage: ''
    }
  },

  mounted() {
    this.initializePuzzle()
  },

  methods: {
    initializePuzzle() {
      this.tiles = [...Array(8).keys()].map(n => n + 1)
      this.tiles.push(null)
      this.moves = 0
      this.completedCurrentLevel = false
      this.levelMessage = ''
      this.shuffleTiles()
    },

    shuffleTiles() {
      do {
        for (let i = this.tiles.length - 1; i > 0; i--) {
          const j = Math.floor(Math.random() * (i + 1))
          ;[this.tiles[i], this.tiles[j]] = [this.tiles[j], this.tiles[i]]
        }
      } while (this.checkWin())

      this.emptyIndex = this.tiles.indexOf(null)
    },

    canMove(index) {
      const row = Math.floor(index / 3)
      const col = index % 3
      const emptyRow = Math.floor(this.emptyIndex / 3)
      const emptyCol = this.emptyIndex % 3

      return (
        (Math.abs(row - emptyRow) === 1 && col === emptyCol) ||
        (Math.abs(col - emptyCol) === 1 && row === emptyRow)
      )
    },

    moveTile(index) {
      if (!this.canMove(index)) return

      ;[this.tiles[index], this.tiles[this.emptyIndex]] = [
        this.tiles[this.emptyIndex],
        this.tiles[index]
      ]

      this.emptyIndex = index
      this.moves++

      if (this.checkWin()) {
        this.handleWin()
      }
    },

    checkWin() {
      for (let i = 0; i < 8; i++) {
        if (this.tiles[i] !== i + 1) return false
      }
      return this.tiles[8] === null
    },

    handleWin() {
      const earned = Math.max(100 - this.moves, 10)
      this.score += earned
      this.lastEarnedScore = earned
      this.showWinModal = true
      this.completedCurrentLevel = true

      if (this.currentLevel + 1 > this.unlockedLevel && this.currentLevel < this.historicalFigures.length - 1) {
        this.unlockedLevel = this.currentLevel + 1
      }
    },

    closeWinModal() {
      this.showWinModal = false
    },

    goToNextLevel() {
      if (!this.completedCurrentLevel) {
        this.levelMessage = '⚠️ You must complete this level before unlocking the next one.'
        return
      }

      if (this.currentLevel < this.historicalFigures.length - 1) {
        this.currentLevel++
        this.initializePuzzle()
        this.levelMessage = ''
      } else {
        this.levelMessage = '🏆 Congratulations! You completed all levels.'
      }
    },

    getTileStyle(tileNumber) {
      const correctIndex = tileNumber - 1
      const row = Math.floor(correctIndex / 3)
      const col = correctIndex % 3

      return {
        backgroundImage: `url(/src/assets/images/${this.historicalFigures[this.currentLevel].image})`,
        backgroundSize: '300px 300px',
        backgroundPosition: `${-col * 100}px ${-row * 100}px`
      }
    }
  }
}
</script>

<style>
body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: linear-gradient(to right, #dbeafe, #fef3c7);
}

.app {
  text-align: center;
  padding: 20px;
}

h1 {
  color: #1e3a8a;
}

.score {
  font-size: 20px;
  font-weight: bold;
}

.board {
  display: grid;
  grid-template-columns: repeat(3, 100px);
  grid-template-rows: repeat(3, 100px);
  gap: 3px;
  justify-content: center;
  margin: 20px auto;
}

.tile {
  width: 100px;
  height: 100px;
  border: 2px solid #000;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 20px;
  cursor: pointer;
  color: white;
  text-shadow: 1px 1px 2px black;
}

.tile.empty {
  background: white;
  cursor: default;
  border: 2px dashed gray;
}

.controls button,
.modal button {
  margin: 5px;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  border: none;
  background: #2563eb;
  color: white;
  border-radius: 8px;
}

.message {
  font-weight: bold;
  color: #b91c1c;
  margin-top: 10px;
}

.levels-container {
  margin-top: 30px;
}

.levels-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 15px;
}

.level-card {
  border: 2px solid #ccc;
  padding: 10px;
  border-radius: 10px;
  background: white;
}

.level-card.active {
  border-color: #2563eb;
}

.level-card.locked {
  opacity: 0.6;
}

.level-card img {
  width: 100%;
  height: 100px;
  object-fit: cover;
  border-radius: 8px;
}

.locked-box {
  height: 100px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #ddd;
  border-radius: 8px;
  font-size: 24px;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background: white;
  padding: 30px;
  border-radius: 12px;
  max-width: 500px;
}
@media (max-width: 1024px) {
  .board {
    grid-template-columns: repeat(3, 90px);
    grid-template-rows: repeat(3, 90px);
  }

  .tile {
    width: 90px;
    height: 90px;
    font-size: 18px;
  }

  .levels-grid {
    grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
  }
}

@media (max-width: 768px) {
  .app {
    padding: 15px;
  }

  h1 {
    font-size: 26px;
  }

  h2 {
    font-size: 20px;
  }

  .score {
    font-size: 18px;
  }

  .board {
    grid-template-columns: repeat(3, 80px);
    grid-template-rows: repeat(3, 80px);
    gap: 2px;
  }

  .tile {
    width: 80px;
    height: 80px;
    font-size: 16px;
  }

  .controls {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .controls button {
    width: 80%;
    margin-bottom: 10px;
  }

  .levels-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  .modal-content {
    width: 90%;
    padding: 20px;
  }
}

@media (max-width: 480px) {
  h1 {
    font-size: 22px;
  }

  h2 {
    font-size: 18px;
  }

  .board {
    grid-template-columns: repeat(3, 65px);
    grid-template-rows: repeat(3, 65px);
  }

  .tile {
    width: 65px;
    height: 65px;
    font-size: 14px;
  }

  .levels-grid {
    grid-template-columns: 1fr;
  }

  .level-card img,
  .locked-box {
    height: 80px;
  }

  .controls button,
  .modal button {
    width: 100%;
    font-size: 14px;
    padding: 10px;
  }

  .score,
  .message {
    font-size: 16px;
  }
}
</style>
