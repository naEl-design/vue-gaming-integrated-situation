<template>
  <div class="whack-game-container">
    <!-- HOME SCREEN -->
    <div v-if="!gameStarted" class="home-screen">
      <div class="game-logo">
        <div class="logo-mole">🐭</div>
        <h1 class="game-title">WHACK-A-MOLE</h1>
        <p class="game-tagline">PRO EDITION</p>
      </div>

      <div class="feature-grid">
        <div class="feature-card">
          <span class="feature-icon">🎮</span>
          <span>50 Challenging Levels</span>
        </div>
        <div class="feature-card">
          <span class="feature-icon">🔊</span>
          <span>Realistic Sound FX</span>
        </div>
        <div class="feature-card">
          <span class="feature-icon">⚡</span>
          <span>Combo System</span>
        </div>
        <div class="feature-card">
          <span class="feature-icon">💾</span>
          <span>Progress Saves</span>
        </div>
      </div>

      <div class="difficulty-section">
        <h3>🎚️ SELECT DIFFICULTY</h3>
        <div class="difficulty-buttons">
          <button 
            :class="['diff-btn', { active: currentDifficulty === 'easy' }]"
            @click="setDifficulty('easy')"
          >
            🌱 EASY
            <span class="diff-desc">Slow moles, +10 pts</span>
          </button>
          <button 
            :class="['diff-btn', { active: currentDifficulty === 'medium' }]"
            @click="setDifficulty('medium')"
          >
            ⚡ MEDIUM
            <span class="diff-desc">Faster, +15 pts</span>
          </button>
          <button 
            :class="['diff-btn', { active: currentDifficulty === 'advanced' }]"
            @click="setDifficulty('advanced')"
          >
            🔥 ADVANCED
            <span class="diff-desc">Very fast, +20 pts</span>
          </button>
        </div>
      </div>

      <div class="level-selector-section">
        <h3>📊 START FROM LEVEL</h3>
        <div class="level-scroll">
          <button 
            v-for="lvl in 50" 
            :key="lvl"
            :class="['level-pill', { 'level-active': currentLevel === lvl }]"
            @click="setLevel(lvl)"
          >
            {{ lvl }}
          </button>
        </div>
      </div>

      <div class="info-panel">
        <div class="info-item">
          <span>🎯 Target Points:</span>
          <strong>{{ requiredPoints }}</strong>
        </div>
        <div class="info-item">
          <span>🏆 Max Combo:</span>
          <strong>{{ savedMaxCombo }}</strong>
        </div>
      </div>

      <button class="start-btn" @click="startGame">
        🚀 START GAME 🚀
      </button>

      <div class="how-to-play">
        <h4>📖 HOW TO PLAY</h4>
        <p>Click on the mole as fast as you can! Each hit gives points based on difficulty.<br>
        Build your combo streak for satisfying sound effects! Complete 50 levels to become champion!</p>
      </div>
    </div>

    <!-- GAME SCREEN -->
    <div v-else class="game-screen">
      <!-- Game Header Stats -->
      <div class="game-header">
        <div class="stat-card">
          <span class="stat-icon">🎯</span>
          <div class="stat-info">
            <span class="stat-label">SCORE</span>
            <span class="stat-value">{{ currentScore }} / {{ requiredPoints }}</span>
          </div>
        </div>
        <div class="stat-card">
          <span class="stat-icon">📊</span>
          <div class="stat-info">
            <span class="stat-label">LEVEL</span>
            <span class="stat-value">{{ currentLevel }} / 50</span>
          </div>
        </div>
        <div class="stat-card">
          <span class="stat-icon">⚡</span>
          <div class="stat-info">
            <span class="stat-label">COMBO</span>
            <span class="stat-value">x{{ combo }}</span>
          </div>
        </div>
        <div class="stat-card">
          <span class="stat-icon">🎚️</span>
          <div class="stat-info">
            <span class="stat-label">DIFFICULTY</span>
            <span class="stat-value">{{ difficultyName }}</span>
          </div>
        </div>
      </div>

      <!-- Game Message with Animation -->
      <div class="message-area" :class="{ 'celebration': showCelebration }">
        {{ gameMessage }}
      </div>

      <!-- 9 Holes Grid -->
      <div class="holes-grid">
        <div 
          v-for="index in 9" 
          :key="index - 1"
          class="hole"
          :data-hole="index - 1"
          @click="handleWhack(index - 1)"
        >
          <div class="hole-shadow"></div>
          <div 
            v-if="activeMole === index - 1" 
            class="mole"
            :class="{ 'mole-hit-animation': hitAnimation }"
          >
            <div class="mole-face">🐭</div>
            <div class="mole-eyes">👀</div>
          </div>
          <div v-else class="empty-hole"></div>
        </div>
      </div>

      <!-- Control Buttons -->
      <div class="game-controls">
        <button class="control-btn home-btn" @click="quitToHome">
          🏠 QUIT TO MENU
        </button>
        <button class="control-btn reset-btn" @click="resetCurrentLevel">
          🔄 RESTART LEVEL
        </button>
      </div>

      <div class="combo-meter" v-if="combo > 0">
        <div class="combo-bar" :style="{ width: Math.min(100, combo * 5) + '%' }"></div>
        <span class="combo-text">🔥 {{ combo }}x COMBO! 🔥</span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'WhackAMoleGame',
  
  data() {
    return {
      // Game state
      gameStarted: false,
      currentDifficulty: 'easy', // easy, medium, advanced
      currentLevel: 1,
      currentScore: 0,
      requiredPoints: 50,
      activeMole: null,
      gameMessage: '🐭 Get ready! Click the mole!',
      showCelebration: false,
      combo: 0,
      maxCombo: 0,
      hitAnimation: false,
      
      // Timers
      moleInterval: null,
      hideTimeout: null,
      isGameActive: false,
      
      // Audio context
      audioContext: null,
      
      // Difficulty settings
      difficultySettings: {
        easy: { intervalMs: 1100, visibleMs: 1000, points: 10, name: 'EASY' },
        medium: { intervalMs: 800, visibleMs: 700, points: 15, name: 'MEDIUM' },
        advanced: { intervalMs: 600, visibleMs: 500, points: 20, name: 'ADVANCED' }
      },
      
      // Save data
      savedMaxCombo: 0
    }
  },
  
  computed: {
    difficultyName() {
      return this.difficultySettings[this.currentDifficulty].name
    }
  },
  
  watch: {
    currentDifficulty() {
      this.updateRequiredPoints()
    },
    currentLevel() {
      this.updateRequiredPoints()
      this.saveProgress()
    }
  },
  
  mounted() {
    this.loadProgress()
    this.updateRequiredPoints()
    this.initAudio()
  },
  
  beforeUnmount() {
    this.stopGameLoop()
    if (this.audioContext) {
      this.audioContext.close()
    }
  },
  
  methods: {
    // ==================== AUDIO SYSTEM ====================
    initAudio() {
      // Audio will be initialized on first user interaction
      window.addEventListener('click', this.resumeAudio, { once: true })
    },
    
    resumeAudio() {
      if (!this.audioContext) {
        this.audioContext = new (window.AudioContext || window.webkitAudioContext)()
      }
      if (this.audioContext.state === 'suspended') {
        this.audioContext.resume()
      }
    },
    
    playHitSound() {
      try {
        if (!this.audioContext) {
          this.audioContext = new (window.AudioContext || window.webkitAudioContext)()
        }
        if (this.audioContext.state === 'suspended') {
          this.audioContext.resume()
        }
        
        const oscillator = this.audioContext.createOscillator()
        const gainNode = this.audioContext.createGain()
        
        oscillator.connect(gainNode)
        gainNode.connect(this.audioContext.destination)
        
        // Higher pitch for higher combo
        const baseFreq = 600 + Math.min(300, this.combo * 20)
        oscillator.frequency.value = baseFreq
        oscillator.type = 'sine'
        
        gainNode.gain.value = 0.3
        oscillator.start()
        gainNode.gain.exponentialRampToValueAtTime(0.0001, this.audioContext.currentTime + 0.3)
        oscillator.stop(this.audioContext.currentTime + 0.3)
      } catch(e) {
        console.warn('Audio error:', e)
      }
    },
    
    playMissSound() {
      try {
        if (!this.audioContext) {
          this.audioContext = new (window.AudioContext || window.webkitAudioContext)()
        }
        if (this.audioContext.state === 'suspended') {
          this.audioContext.resume()
        }
        
        const oscillator = this.audioContext.createOscillator()
        const gainNode = this.audioContext.createGain()
        
        oscillator.connect(gainNode)
        gainNode.connect(this.audioContext.destination)
        
        oscillator.frequency.value = 300
        oscillator.type = 'triangle'
        
        gainNode.gain.value = 0.2
        oscillator.start()
        gainNode.gain.exponentialRampToValueAtTime(0.0001, this.audioContext.currentTime + 0.2)
        oscillator.stop(this.audioContext.currentTime + 0.2)
      } catch(e) {}
    },
    
    playLevelUpSound() {
      try {
        if (!this.audioContext) {
          this.audioContext = new (window.AudioContext || window.webkitAudioContext)()
        }
        if (this.audioContext.state === 'suspended') {
          this.audioContext.resume()
        }
        
        const playNote = (freq, delay = 0) => {
          const osc = this.audioContext.createOscillator()
          const gain = this.audioContext.createGain()
          osc.connect(gain)
          gain.connect(this.audioContext.destination)
          osc.frequency.value = freq
          osc.type = 'sine'
          gain.gain.value = 0.3
          const startTime = this.audioContext.currentTime + delay
          osc.start(startTime)
          gain.gain.exponentialRampToValueAtTime(0.0001, startTime + 0.3)
          osc.stop(startTime + 0.3)
        }
        
        playNote(523.25)  // C5
        playNote(659.25, 0.15) // E5
        playNote(783.99, 0.3)  // G5
      } catch(e) {}
    },
    
    // ==================== GAME LOGIC ====================
    updateRequiredPoints() {
      const difficulty = this.currentDifficulty
      const level = this.currentLevel
      
      if (difficulty === 'easy') {
        this.requiredPoints = 50 + (level - 1) * 8
      } else if (difficulty === 'medium') {
        this.requiredPoints = 70 + (level - 1) * 12
      } else {
        this.requiredPoints = 100 + (level - 1) * 18
      }
    },
    
    showMole() {
      if (!this.isGameActive || !this.gameStarted) return
      
      // Clear previous hide timeout
      if (this.hideTimeout) clearTimeout(this.hideTimeout)
      
      // Random hole 0-8
      const randomHole = Math.floor(Math.random() * 9)
      this.activeMole = randomHole
      
      // Auto hide after visible duration
      const settings = this.difficultySettings[this.currentDifficulty]
      this.hideTimeout = setTimeout(() => {
        if (this.isGameActive && this.gameStarted && this.activeMole !== null) {
          this.activeMole = null
          // Break combo on missed mole
          if (this.combo > 0) {
            this.combo = 0
            this.gameMessage = '💨 Missed! Combo broken!'
            setTimeout(() => {
              if (this.isGameActive) this.gameMessage = '🎯 Keep whacking!'
            }, 800)
          }
        }
      }, settings.visibleMs)
    },
    
    handleWhack(holeIndex) {
      if (!this.gameStarted || !this.isGameActive) return
      
      if (this.activeMole === holeIndex) {
        // HIT!
        this.playHitSound()
        
        // Hit animation
        this.hitAnimation = true
        setTimeout(() => { this.hitAnimation = false }, 150)
        
        const settings = this.difficultySettings[this.currentDifficulty]
        const pointsEarned = settings.points
        this.currentScore += pointsEarned
        this.combo += 1
        
        // Update max combo
        if (this.combo > this.maxCombo) {
          this.maxCombo = this.combo
          this.savedMaxCombo = this.maxCombo
          this.saveProgress()
        }
        
        this.gameMessage = `💥 WHACK! +${pointsEarned} pts | COMBO: x${this.combo} 🔥`
        
        // Hide mole immediately
        if (this.hideTimeout) clearTimeout(this.hideTimeout)
        this.activeMole = null
        
        // Check level completion
        if (this.currentScore >= this.requiredPoints) {
          this.completeLevel()
        } else {
          setTimeout(() => {
            if (this.isGameActive && this.gameMessage.includes('WHACK')) {
              this.gameMessage = '🐭 Keep going!'
            }
          }, 800)
        }
      } else if (this.activeMole !== null) {
        // Wrong hole - miss
        this.playMissSound()
        this.combo = 0
        this.gameMessage = '❌ Missed! Click on the mole!'
        setTimeout(() => {
          if (this.isGameActive) this.gameMessage = '🔨 Focus!'
        }, 700)
      }
    },
    
    completeLevel() {
      this.playLevelUpSound()
      
      if (this.currentLevel < 50) {
        // Level up!
        this.currentLevel++
        // Carry over excess points
        const excess = this.currentScore - this.requiredPoints
        this.currentScore = excess > 0 ? excess : 0
        this.updateRequiredPoints()
        
        this.showCelebration = true
        this.gameMessage = `✨✨ LEVEL UP! Now at Level ${this.currentLevel} ✨✨\n🎯 Need ${this.requiredPoints} points!`
        
        setTimeout(() => {
          this.showCelebration = false
          if (this.isGameActive) {
            this.gameMessage = `🌟 Level ${this.currentLevel} - Let's go! 🌟`
          }
        }, 2000)
        
        this.saveProgress()
      } else {
        // Completed all 50 levels!
        this.gameMessage = '🏆🏆 GRAND CHAMPION! You completed all 50 levels! 🏆🏆'
        this.showCelebration = true
        this.stopGameLoop()
        this.isGameActive = false
        this.gameStarted = false
        
        setTimeout(() => {
          this.showCelebration = false
        }, 3000)
      }
    },
    
    // ==================== GAME CONTROL ====================
    startGameLoop() {
      if (this.moleInterval) clearInterval(this.moleInterval)
      
      const settings = this.difficultySettings[this.currentDifficulty]
      this.moleInterval = setInterval(() => {
        if (this.gameStarted && this.isGameActive) {
          this.showMole()
        }
      }, settings.intervalMs)
    },
    
    stopGameLoop() {
      if (this.moleInterval) {
        clearInterval(this.moleInterval)
        this.moleInterval = null
      }
      if (this.hideTimeout) {
        clearTimeout(this.hideTimeout)
        this.hideTimeout = null
      }
    },
    
    startGame() {
      this.stopGameLoop()
      
      // Reset game state
      this.currentScore = 0
      this.combo = 0
      this.activeMole = null
      this.updateRequiredPoints()
      
      this.gameMessage = `🎮 GAME START! Difficulty: ${this.difficultyName} | Level ${this.currentLevel} | Target: ${this.requiredPoints} pts`
      this.gameStarted = true
      this.isGameActive = true
      
      this.startGameLoop()
    },
    
    quitToHome() {
      this.stopGameLoop()
      this.gameStarted = false
      this.isGameActive = false
      this.activeMole = null
      this.saveProgress()
    },
    
    resetCurrentLevel() {
      if (!this.gameStarted) return
      
      this.currentScore = 0
      this.combo = 0
      this.activeMole = null
      this.updateRequiredPoints()
      this.gameMessage = '🔄 Level restarted! Good luck!'
      
      // Restart loop to ensure fresh state
      this.stopGameLoop()
      this.startGameLoop()
    },
    
    // ==================== PROGRESS SAVE/LOAD ====================
    saveProgress() {
      const progress = {
        currentDifficulty: this.currentDifficulty,
        currentLevel: this.currentLevel,
        maxCombo: this.maxCombo,
        savedMaxCombo: this.savedMaxCombo
      }
      localStorage.setItem('whackMoleProgress', JSON.stringify(progress))
    },
    
    loadProgress() {
      const saved = localStorage.getItem('whackMoleProgress')
      if (saved) {
        try {
          const progress = JSON.parse(saved)
          this.currentDifficulty = progress.currentDifficulty || 'easy'
          this.currentLevel = progress.currentLevel || 1
          this.maxCombo = progress.maxCombo || 0
          this.savedMaxCombo = progress.savedMaxCombo || 0
          this.updateRequiredPoints()
        } catch(e) {}
      }
    },
    
    // ==================== HOME SCREEN METHODS ====================
    setDifficulty(diff) {
      if (!this.gameStarted) {
        this.currentDifficulty = diff
        this.currentLevel = 1
        this.currentScore = 0
        this.combo = 0
        this.updateRequiredPoints()
        this.saveProgress()
      }
    },
    
    setLevel(level) {
      if (!this.gameStarted) {
        this.currentLevel = level
        this.currentScore = 0
        this.combo = 0
        this.updateRequiredPoints()
        this.saveProgress()
      }
    }
  }
}
</script>

<style scoped>
/* Global Container */
.whack-game-container {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  font-family: 'Segoe UI', 'Inter', system-ui, -apple-system, sans-serif;
  background: linear-gradient(135deg, #1a472a 0%, #0e2a1a 100%);
  padding: 20px;
}

/* ==================== HOME SCREEN ==================== */
.home-screen {
  max-width: 900px;
  width: 100%;
  background: rgba(30, 25, 20, 0.85);
  backdrop-filter: blur(16px);
  border-radius: 60px;
  padding: 40px 30px;
  text-align: center;
  box-shadow: 0 25px 50px -12px rgba(0,0,0,0.5), inset 0 1px 0 rgba(255,255,255,0.1);
  border: 1px solid rgba(255, 200, 100, 0.3);
}

.game-logo {
  margin-bottom: 30px;
}

.logo-mole {
  font-size: 80px;
  animation: bounce 2s infinite;
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}

.game-title {
  font-size: 48px;
  font-weight: 800;
  background: linear-gradient(135deg, #FFE6B0, #FFB347);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  letter-spacing: -0.02em;
  margin: 10px 0 5px;
}

.game-tagline {
  color: #ffd966;
  font-weight: 600;
  letter-spacing: 4px;
  font-size: 14px;
}

.feature-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  gap: 15px;
  margin: 30px 0;
}

.feature-card {
  background: rgba(0,0,0,0.4);
  padding: 12px;
  border-radius: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  color: #ffe2b5;
  font-weight: 600;
}

.feature-icon {
  font-size: 24px;
}

.difficulty-section {
  margin: 25px 0;
}

.difficulty-section h3 {
  color: #ffd966;
  margin-bottom: 15px;
}

.difficulty-buttons {
  display: flex;
 gap: 15px;
  justify-content: center;
  flex-wrap: wrap;
}

.diff-btn {
  background: #3a2a1e;
  border: none;
  padding: 12px 24px;
  border-radius: 50px;
  font-family: inherit;
  font-weight: bold;
  font-size: 16px;
  color: #ffdead;
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
}

.diff-btn.active {
  background: #f5a623;
  color: #1e2a1c;
  box-shadow: 0 0 15px #ffbb44;
  transform: scale(1.02);
}

.diff-desc {
  font-size: 11px;
  opacity: 0.8;
}

.level-selector-section {
  margin: 25px 0;
}

.level-selector-section h3 {
  color: #ffd966;
  margin-bottom: 15px;
}

.level-scroll {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 8px;
  max-height: 160px;
  overflow-y: auto;
  padding: 10px;
  background: rgba(0,0,0,0.3);
  border-radius: 30px;
}

.level-pill {
  background: #2a2018;
  border: none;
  width: 45px;
  padding: 8px 0;
  border-radius: 30px;
  font-weight: bold;
  color: #ffde9c;
  cursor: pointer;
  transition: 0.1s;
}

.level-pill:hover {
  background: #f5a623;
  color: #1a1a1a;
}

.level-active {
  background: #f5a623;
  color: #1e2a1c;
  box-shadow: 0 0 8px #ffbb44;
}

.info-panel {
  display: flex;
  justify-content: center;
  gap: 30px;
  background: rgba(0,0,0,0.4);
  padding: 15px;
  border-radius: 50px;
  margin: 20px 0;
}

.info-item {
  display: flex;
  gap: 8px;
  color: #ffe2b5;
  font-weight: 600;
}

.info-item strong {
  color: #f5a623;
  font-size: 20px;
}

.start-btn {
  background: #f5a623;
  border: none;
  font-size: 28px;
  font-weight: 800;
  padding: 15px 40px;
  border-radius: 60px;
  font-family: inherit;
  cursor: pointer;
  transition: 0.1s linear;
  box-shadow: 0 10px 0 #a45817;
  margin: 20px 0;
}

.start-btn:active {
  transform: translateY(4px);
  box-shadow: 0 6px 0 #a45817;
}

.how-to-play {
  background: rgba(0,0,0,0.4);
  border-radius: 30px;
  padding: 15px;
  margin-top: 20px;
}

.how-to-play h4 {
  color: #ffd966;
  margin-bottom: 8px;
}

.how-to-play p {
  color: #ddc;
  font-size: 14px;
  line-height: 1.5;
}

/* ==================== GAME SCREEN ==================== */
.game-screen {
  max-width: 750px;
  width: 100%;
  background: rgba(35, 45, 30, 0.9);
  backdrop-filter: blur(12px);
  border-radius: 60px;
  padding: 25px;
  box-shadow: 0 20px 40px rgba(0,0,0,0.4);
  border: 1px solid rgba(255,200,100,0.3);
}

.game-header {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 12px;
  margin-bottom: 25px;
}

.stat-card {
  background: #1a1f14;
  border-radius: 30px;
  padding: 10px;
  display: flex;
  align-items: center;
  gap: 8px;
  justify-content: center;
}

.stat-icon {
  font-size: 24px;
}

.stat-info {
  display: flex;
  flex-direction: column;
}

.stat-label {
  font-size: 10px;
  color: #ffd966;
  letter-spacing: 1px;
}

.stat-value {
  font-weight: 800;
  font-size: 16px;
  color: white;
}

.message-area {
  background: #000000aa;
  backdrop-filter: blur(8px);
  border-radius: 50px;
  padding: 12px;
  text-align: center;
  font-weight: bold;
  font-size: 16px;
  color: #ffde9c;
  margin-bottom: 25px;
  transition: all 0.3s;
}

.message-area.celebration {
  background: #f5a623cc;
  color: #1e2a1c;
  transform: scale(1.02);
  font-size: 18px;
}

.holes-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  justify-items: center;
  margin: 25px 0;
}

.hole {
  width: 100%;
  aspect-ratio: 1 / 1;
  max-width: 120px;
  position: relative;
  cursor: pointer;
}

.hole-shadow {
  position: absolute;
  width: 100%;
  height: 100%;
  background: radial-gradient(ellipse at 30% 30%, #4a3a28, #1f160e);
  border-radius: 50%;
  box-shadow: inset 0 8px 12px rgba(0,0,0,0.6), 0 8px 15px rgba(0,0,0,0.4);
}

.empty-hole {
  width: 80%;
  height: 80%;
  background: #2a1f12;
  border-radius: 50%;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  opacity: 0.6;
}

.mole {
  position: absolute;
  width: 90%;
  height: 90%;
  background: radial-gradient(circle at 35% 30%, #d68f3c, #8b5a2b);
  border-radius: 50% 50% 45% 45%;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  cursor: pointer;
  transition: 0.05s linear;
  box-shadow: 0 6px 0 #5a3a1e;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.mole.mole-hit-animation {
  animation: whackShake 0.12s ease-in-out 2;
}

@keyframes whackShake {
  0% { transform: translate(-50%, -50%) rotate(0deg); }
  50% { transform: translate(-45%, -55%) rotate(-8deg); }
  100% { transform: translate(-50%, -50%) rotate(0deg); }
}

.mole-face {
  font-size: 45px;
  filter: drop-shadow(2px 4px 4px rgba(0,0,0,0.4));
}

.mole-eyes {
  font-size: 18px;
  position: absolute;
  top: 35%;
  left: 50%;
  transform: translateX(-50%);
  white-space: nowrap;
}

.game-controls {
  display: flex;
  gap: 15px;
  justify-content: center;
  margin: 20px 0;
}

.control-btn {
  padding: 10px 24px;
  border-radius: 40px;
  font-family: inherit;
  font-weight: bold;
  border: none;
  cursor: pointer;
  transition: 0.1s;
  font-size: 14px;
}

.home-btn {
  background: #5a4a3a;
  color: #ffdead;
}

.reset-btn {
  background: #8b5a2b;
  color: #ffefc0;
}

.control-btn:active {
  transform: scale(0.96);
}

.combo-meter {
  margin-top: 15px;
  background: #2a2018;
  border-radius: 30px;
  overflow: hidden;
  position: relative;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.combo-bar {
  position: absolute;
  left: 0;
  top: 0;
  height: 100%;
  background: linear-gradient(90deg, #f5a623, #ff6b35);
  transition: width 0.2s ease;
  opacity: 0.6;
}

.combo-text {
  position: relative;
  z-index: 1;
  font-weight: 800;
  color: #fff2d0;
  text-shadow: 0 1px 2px black;
}

/* Responsive */
@media (max-width: 550px) {
  .game-header {
    grid-template-columns: repeat(2, 1fr);
  }
  .holes-grid {
    gap: 12px;
  }
  .mole-face {
    font-size: 30px;
  }
  .game-title {
    font-size: 32px;
  }
  .start-btn {
    font-size: 22px;
    padding: 12px 25px;
  }
}
</style>