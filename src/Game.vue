<template>
  <div class="game-container">
    <canvas ref="gameCanvas" width="800" height="600"></canvas>
    
    <!-- HUD -->
    <div class="hud">
      <div class="hud-item">FUEL: {{ Math.max(0, fuel) }}</div>
      <div class="hud-item">ALTITUDE: {{ Math.max(0, Math.floor(550 - y)) }}</div>
      <div class="hud-item">V.SPEED: {{ (-vy).toFixed(1) }}</div>
      <div class="hud-item">H.SPEED: {{ vx.toFixed(1) }}</div>
      <div class="hud-item">ANGLE: {{ angle.toFixed(0) }}°</div>
    </div>
    
    <!-- Controls Help -->
    <div class="controls">
      <div>W/↑: THRUST</div>
      <div>A/←: ROTATE LEFT</div>
      <div>D/→: ROTATE RIGHT</div>
      <div>R: RESTART</div>
      <div>SPACE: START / RESTART</div>
    </div>
    
    <!-- Game Over Overlay -->
    <div v-if="gameState === 'won'" class="overlay won">
      <div class="overlay-content">
        <h1>🎉 {{ winMessage.title }} 🎉</h1>
        <p class="compliment">{{ winMessage.message }}</p>
        <img :src="winMessage.imageUrl" alt="Good Boy Meme" class="meme-image" />
        <button @click="restartGame" class="overlay-button">PLAY AGAIN</button>
        <p class="key-hint">Press SPACE to restart</p>
      </div>
    </div>
    
    <div v-if="gameState === 'lost'" class="overlay lost">
      <div class="overlay-content">
        <h1>💥 CRASHED! 💥</h1>
        <p>{{ crashReason }}</p>
        <button @click="restartGame" class="overlay-button">TRY AGAIN</button>
        <p class="key-hint">Press SPACE to restart</p>
      </div>
    </div>
    
    <!-- Start Screen -->
    <div v-if="gameState === 'start'" class="overlay start">
      <div class="overlay-content">
        <h1>🌙 MOON LANDING 🚀</h1>
        <p>Land gently on the platform!</p>
        <p class="requirements">
          ✓ Vertical speed < 2.0<br/>
          ✓ Angle near 0°<br/>
          ✓ Don't run out of fuel!
        </p>
        <button @click="startGame" class="overlay-button">START MISSION</button>
        <p class="key-hint">Press SPACE to start</p>
      </div>
    </div>
  </div>
</template>

<script>
// Audio assets (royalty-free from Pixabay/FreeSound)
const AUDIO_ASSETS = {
  background: 'https://cdn.pixabay.com/audio/2022/03/24/audio_07d931d38c.mp3',
  thrust: 'https://cdn.pixabay.com/audio/2022/03/10/audio_32c7a5d76c.mp3',
  landingSoft: 'https://cdn.pixabay.com/audio/2022/03/15/audio_c8c8a23f6d.mp3',
  landingCrash: 'https://cdn.pixabay.com/audio/2022/03/09/audio_145f9e0e2a.mp3'
}

// Hilarious Doge/Good Boy compliments for successful landings
const WIN_COMPLIMENTS = [
  { title: 'SENSATIONAL LANDING!', message: "You're a very good boy! 🐕✨", imageUrl: 'https://images.unsplash.com/photo-1583337130417-3346a1be7dee?w=400&h=400&fit=crop' },
  { title: 'LUNAR LEGEND!', message: 'Much moon, very landing! 🚀🌕', imageUrl: 'https://images.unsplash.com/photo-1561037404-61cd46aa615b?w=400&h=400&fit=crop' },
  { title: 'WOW! SUCH SKILL!', message: 'NASA is calling... they want their astronaut back! 🐕👨‍🚀', imageUrl: 'https://images.unsplash.com/photo-1517849845537-4d257902454a?w=400&h=400&fit=crop' },
  { title: 'PERFECT TOUCHDOWN!', message: 'Gentle like a feather, precise like a... good boy! 🪶🐕', imageUrl: 'https://images.unsplash.com/photo-1534351590666-13e3e96b5017?w=400&h=400&fit=crop' },
  { title: 'MOON MASTER!', message: 'The moon now has a new favorite visitor! 🌙👑', imageUrl: 'https://images.unsplash.com/photo-1587300003388-59208cc962cb?w=400&h=400&fit=crop' },
  { title: 'INCREDIBLE!', message: 'That landing was so smooth, even the moon is impressed! 🌕😍', imageUrl: 'https://images.unsplash.com/photo-1591946614720-885a27ea4e65?w=400&h=400&fit=crop' },
  { title: 'SPACE ACE!', message: 'Zero crashes, 100% good boy! 🏆🐕', imageUrl: 'https://images.unsplash.com/photo-1552053831-71594a27632d?w=400&h=400&fit=crop' },
  { title: 'LEGENDARY!', message: 'Elon Musk wishes he could land like you! 🚀💪', imageUrl: 'https://images.unsplash.com/photo-1589985270826-4b7bb135bc9d?w=400&h=400&fit=crop' },
  { title: 'ASTRO-DOG SUPREME!', message: 'The galaxy applauds your landing skills! 🌌👏', imageUrl: 'https://images.unsplash.com/photo-1537151608828-ea2b11777ee8?w=400&h=400&fit=crop' },
  { title: 'FLAWLESS VICTORY!', message: 'Like a butterfly, sting like a... wait, you are a good boy! 🦋🐕', imageUrl: 'https://images.unsplash.com/photo-1518331483807-f6adc0e1ad23?w=400&h=400&fit=crop' },
  { title: 'COSMIC GOOD BOY!', message: 'The universe itself bows to your landing prowess! 🌠🐕', imageUrl: 'https://images.unsplash.com/photo-1530281700549-e82e7bf110d6?w=400&h=400&fit=crop' },
  { title: 'GALACTIC HERO!', message: 'Shiba Inu: 1, Gravity: 0! 🐕💫', imageUrl: 'https://images.unsplash.com/photo-1548199973-03cce0bbc87b?w=400&h=400&fit=crop' }
]

export default {
  name: 'Game',
  data() {
    return {
      // Physics state
      x: 400,
      y: 100,
      vx: 0,
      vy: 0,
      angle: 0,
      angularVelocity: 0,
      fuel: 100,
      
      // Game constants
      gravity: 0.04,
      thrustPower: 0.18,
      rotationSpeed: 0.015,
      friction: 0.999,
      angularFriction: 0.98,
      
      // Landing platform
      platformX: 350,
      platformY: 550,
      platformWidth: 100,
      
      // Game state
      gameState: 'start',
      crashReason: '',
      winMessage: null,
      
      // Input state
      keys: {
        up: false,
        left: false,
        right: false
      },
      
      // Animation
      animationId: null,
      lastTime: 0,
      
      // Audio system
      audio: {
        background: null,
        thrust: null,
        landingSoft: null,
        landingCrash: null,
        initialized: false,
        thrustPlaying: false
      }
    }
  },
  
  mounted() {
    this.canvas = this.$refs.gameCanvas
    this.ctx = this.canvas.getContext('2d')
    
    // Input handlers
    window.addEventListener('keydown', this.handleKeyDown)
    window.addEventListener('keyup', this.handleKeyUp)
    
    // Initialize audio on first user interaction
    this.initAudio()
    
    // Start render loop
    this.gameLoop()
  },
  
  beforeUnmount() {
    window.removeEventListener('keydown', this.handleKeyDown)
    window.removeEventListener('keyup', this.handleKeyUp)
    if (this.animationId) {
      cancelAnimationFrame(this.animationId)
    }
    this.cleanupAudio()
  },
  
  methods: {
    initAudio() {
      if (this.audio.initialized) return
      try {
        this.audio.background = new Audio(AUDIO_ASSETS.background)
        this.audio.background.loop = true
        this.audio.background.volume = 0.3
        this.audio.thrust = new Audio(AUDIO_ASSETS.thrust)
        this.audio.thrust.loop = true
        this.audio.thrust.volume = 0.4
        this.audio.landingSoft = new Audio(AUDIO_ASSETS.landingSoft)
        this.audio.landingSoft.volume = 0.5
        this.audio.landingCrash = new Audio(AUDIO_ASSETS.landingCrash)
        this.audio.landingCrash.volume = 0.6
        this.audio.initialized = true
        console.log('Audio initialized successfully')
      } catch (e) { console.error('Audio init failed:', e) }
    },
    
    cleanupAudio() {
      if (this.audio.background) {
        this.audio.background.pause()
        this.audio.background = null
      }
      if (this.audio.thrust) {
        this.audio.thrust.pause()
        this.audio.thrust = null
      }
    },
    
    startBackgroundMusic() {
      if (this.audio.background && !this.audio.background.paused) {
        return
      }
      this.audio.background.play().catch(err => {
        console.log('Background music autoplay blocked:', err)
      })
    },
    
    stopBackgroundMusic() {
      if (this.audio.background) {
        this.audio.background.pause()
        this.audio.background.currentTime = 0
      }
    },
    
    playThrustSound() {
      if (this.audio.thrust && !this.audio.thrustPlaying) {
        this.audio.thrustPlaying = true
        this.audio.thrust.play().catch(err => console.log('Thrust SFX blocked:', err))
      }
    },
    
    stopThrustSound() {
      if (this.audio.thrust && this.audio.thrustPlaying) {
        this.audio.thrustPlaying = false
        this.audio.thrust.pause()
        this.audio.thrust.currentTime = 0
      }
    },
    
    playLandingSound(success) {
      if (success) {
        this.audio.landingSoft.play().catch(err => console.log('Soft landing SFX blocked:', err))
      } else {
        this.audio.landingCrash.play().catch(err => console.log('Crash SFX blocked:', err))
      }
    },
    
    handleKeyDown(e) {
      // Game controls
      if (e.key === 'w' || e.key === 'W' || e.key === 'ArrowUp') {
        this.keys.up = true
      }
      if (e.key === 'a' || e.key === 'A' || e.key === 'ArrowLeft') {
        this.keys.left = true
      }
      if (e.key === 'd' || e.key === 'D' || e.key === 'ArrowRight') {
        this.keys.right = true
      }
      if (e.key === 'r' || e.key === 'R') {
        this.restartGame()
      }
      
      // Space bar for starting/restarting (full keyboard accessibility)
      if (e.key === ' ' || e.code === 'Space') {
        e.preventDefault() // Prevent page scrolling
        if (this.gameState === 'start') {
          this.startGame()
        } else if (this.gameState === 'won' || this.gameState === 'lost') {
          this.restartGame()
        }
      }
    },
    
    handleKeyUp(e) {
      if (e.key === 'w' || e.key === 'W' || e.key === 'ArrowUp') {
        this.keys.up = false
      }
      if (e.key === 'a' || e.key === 'A' || e.key === 'ArrowLeft') {
        this.keys.left = false
      }
      if (e.key === 'd' || e.key === 'D' || e.key === 'ArrowRight') {
        this.keys.right = false
      }
    },
    
    startGame() {
      this.initAudio()
      this.startBackgroundMusic()
      this.gameState = 'playing'
      this.resetPhysics()
    },
    
    restartGame() {
      this.stopBackgroundMusic()
      this.gameState = 'start'
      this.resetPhysics()
    },
    
    resetPhysics() {
      this.x = 400 + (Math.random() - 0.5) * 200
      this.y = 50 + Math.random() * 50
      this.vx = (Math.random() - 0.5) * 2
      this.vy = 0
      this.angle = (Math.random() - 0.5) * 0.5
      this.angularVelocity = 0
      this.fuel = 100
      this.stopThrustSound()
    },
    
    updatePhysics() {
      if (this.gameState !== 'playing') return
      
      // Rotation control
      if (this.keys.left) {
        this.angularVelocity -= this.rotationSpeed
      }
      if (this.keys.right) {
        this.angularVelocity += this.rotationSpeed
      }
      
      // Apply angular friction
      this.angularVelocity *= this.angularFriction
      this.angle += this.angularVelocity
      
      // Thrust with audio feedback
      if (this.keys.up && this.fuel > 0) {
        this.vx += Math.sin(this.angle) * this.thrustPower
        this.vy -= Math.cos(this.angle) * this.thrustPower
        this.fuel -= 0.3
        this.playThrustSound()
      } else {
        this.stopThrustSound()
      }
      
      // Gravity
      this.vy += this.gravity
      
      // Apply friction
      this.vx *= this.friction
      
      // Update position
      this.x += this.vx
      this.y += this.vy
      
      // Screen wrapping (horizontal)
      if (this.x < 0) this.x = 800
      if (this.x > 800) this.x = 0
      
      // Check landing
      this.checkLanding()
    },
    
    checkLanding() {
      if (this.gameState !== 'playing') return
      if (this.y + 10 >= this.platformY) {
        const onPlatform = this.x >= this.platformX && this.x <= this.platformX + this.platformWidth
        if (!onPlatform) {
          this.gameState = 'lost'
          this.crashReason = 'Missed the landing platform!'
          this.playLandingSound(false)
          this.stopThrustSound()
          return
        }
        let normalizedAngle = this.angle % (Math.PI * 2)
        if (normalizedAngle > Math.PI) normalizedAngle -= Math.PI * 2
        if (normalizedAngle < -Math.PI) normalizedAngle += Math.PI * 2
        const angleDegrees = Math.abs(normalizedAngle * 180 / Math.PI)
        const verticalSpeed = Math.abs(this.vy)
        const horizontalSpeed = Math.abs(this.vx)
        const isUpright = angleDegrees < 20
        if (verticalSpeed < 3.5 && horizontalSpeed < 2.5 && isUpright) {
          this.gameState = 'won'
          this.winMessage = WIN_COMPLIMENTS[Math.floor(Math.random() * WIN_COMPLIMENTS.length)]
          this.playLandingSound(true)
          this.stopThrustSound()
          this.stopBackgroundMusic()
        } else {
          this.gameState = 'lost'
          this.playLandingSound(false)
          this.stopThrustSound()
          if (verticalSpeed >= 3.5) this.crashReason = `Too fast! Vertical: ${verticalSpeed.toFixed(1)} (max: 3.5)`
          else if (horizontalSpeed >= 2.5) this.crashReason = `Too much drift! Horizontal: ${horizontalSpeed.toFixed(1)} (max: 2.5)`
          else if (!isUpright) this.crashReason = `Tilted too much! ${angleDegrees.toFixed(0)}° (max: 20°)`
        }
      }
      if (this.y < 0) { this.y = 0; this.vy = 0 }
    },
    
    draw() {
      const ctx = this.ctx
      
      // Clear canvas
      ctx.fillStyle = '#0a0a20'
      ctx.fillRect(0, 0, 800, 600)
      
      // Draw stars
      ctx.fillStyle = '#fff'
      for (let i = 0; i < 50; i++) {
        const x = (i * 37) % 800
        const y = (i * 23) % 400
        ctx.beginPath()
        ctx.arc(x, y, Math.random() * 1.5, 0, Math.PI * 2)
        ctx.fill()
      }
      
      // Draw moon surface
      ctx.fillStyle = '#2a2a2a'
      ctx.fillRect(0, 550, 800, 50)
      
      // Draw landing platform
      ctx.fillStyle = '#4a4a6a'
      ctx.fillRect(this.platformX, this.platformY, this.platformWidth, 10)
      
      // Platform markers
      ctx.fillStyle = '#6a6a8a'
      ctx.fillRect(this.platformX, this.platformY - 5, 5, 15)
      ctx.fillRect(this.platformX + this.platformWidth - 5, this.platformY - 5, 5, 15)
      
      // Draw lander
      ctx.save()
      ctx.translate(this.x, this.y)
      ctx.rotate(this.angle)
      
      // Lander body
      ctx.fillStyle = '#ccc'
      ctx.beginPath()
      ctx.moveTo(0, -15)
      ctx.lineTo(10, 5)
      ctx.lineTo(0, 10)
      ctx.lineTo(-10, 5)
      ctx.closePath()
      ctx.fill()
      
      // Lander legs
      ctx.strokeStyle = '#999'
      ctx.lineWidth = 2
      ctx.beginPath()
      ctx.moveTo(-8, 8)
      ctx.lineTo(-12, 15)
      ctx.moveTo(8, 8)
      ctx.lineTo(12, 15)
      ctx.stroke()
      
      // Thrust flame
      if (this.keys.up && this.fuel > 0 && this.gameState === 'playing') {
        ctx.fillStyle = '#ff6600'
        ctx.beginPath()
        ctx.moveTo(-5, 12)
        ctx.lineTo(0, 25 + Math.random() * 10)
        ctx.lineTo(5, 12)
        ctx.closePath()
        ctx.fill()
      }
      
      ctx.restore()
      
      // Draw fuel warning
      if (this.fuel < 20 && this.gameState === 'playing') {
        ctx.fillStyle = '#ff0000'
        ctx.font = 'bold 16px Courier New'
        ctx.fillText('⚠️ LOW FUEL', 10, 30)
      }
    },
    
    gameLoop(timestamp) {
      this.updatePhysics()
      this.draw()
      this.animationId = requestAnimationFrame((t) => this.gameLoop(t))
    }
  }
}
</script>

<style scoped>
.game-container {
  position: relative;
  width: 800px;
  height: 600px;
  margin: 20px auto;
  border: 2px solid #4a4a6a;
  border-radius: 8px;
  overflow: hidden;
}

canvas {
  display: block;
  background: #000;
}

.hud {
  position: absolute;
  top: 10px;
  left: 10px;
  color: #0f0;
  font-family: 'Courier New', monospace;
  font-size: 14px;
  text-shadow: 0 0 5px #0f0;
}

.hud-item {
  margin-bottom: 5px;
}

.controls {
  position: absolute;
  top: 10px;
  right: 10px;
  color: #aaa;
  font-family: 'Courier New', monospace;
  font-size: 12px;
  text-align: right;
}

.controls div {
  margin-bottom: 3px;
}

.overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  backdrop-filter: blur(5px);
}

.overlay.start {
  background: rgba(0, 0, 0, 0.8);
}

.overlay.won {
  background: rgba(0, 50, 0, 0.9);
}

.overlay.lost {
  background: rgba(50, 0, 0, 0.9);
}

.overlay-content {
  text-align: center;
  color: #fff;
  padding: 40px;
  background: rgba(0, 0, 0, 0.7);
  border-radius: 10px;
  border: 2px solid #4a4a6a;
}

.overlay-content h1 {
  font-size: 32px;
  margin-bottom: 20px;
}

.overlay-content p {
  font-size: 16px;
  margin-bottom: 15px;
}

.requirements {
  text-align: left;
  margin: 20px auto;
  display: inline-block;
}

.compliment {
  font-size: 18px;
  font-style: italic;
  color: #ffd700;
  margin: 10px 0 20px;
}

.meme-image {
  max-width: 300px;
  max-height: 300px;
  border-radius: 10px;
  margin: 20px 0;
  border: 3px solid #fff;
}

.overlay-button {
  display: block;
  margin: 20px auto 0;
  background: #4a4a6a;
  color: #fff;
  border: none;
  padding: 15px 30px;
  font-size: 16px;
  font-family: 'Courier New', monospace;
  cursor: pointer;
  border-radius: 5px;
  transition: background 0.3s;
}

.overlay-button:hover {
  background: #6a6a8a;
}

.key-hint {
  margin-top: 15px;
  font-size: 14px;
  color: #888;
  font-style: italic;
}

.key-hint::before {
  content: '⌨️ ';
}
</style>

<style>
body {
  margin: 0;
  padding: 0;
  background-color: #0a0a20;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  color: white;
  font-family: 'Courier New', monospace;
}
</style>
