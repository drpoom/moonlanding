# 🌙 Moon Landing Game

A classic lunar lander game built with Vue 3 and Vite.

## 🚀 Quick Start

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build
```

## 🎮 Controls

- **W / ↑** - Thrust (uses fuel)
- **A / ←** - Rotate left
- **D / →** - Rotate right
- **R** - Restart game

## 🎯 Objective

Land your spacecraft gently on the marked platform:

- ✅ Vertical speed must be < 2.0
- ✅ Landing angle must be near 0° (±15°)
- ✅ Must land on the platform (not the surrounding terrain)

## 🏆 Success

Land successfully and you'll be rewarded with a hilarious Shiba Inu "Good Boy" meme! 🐕

## 📁 Project Structure

```
moon-landing/
├── index.html          # Entry HTML
├── package.json        # Dependencies
├── vite.config.js      # Vite configuration
└── src/
    ├── main.js         # Vue app entry
    └── Game.vue        # Single-file game component
```

## 🛠️ Tech Stack

- Vue 3 (Composition API)
- Vite 5
- HTML5 Canvas
- Vanilla JavaScript physics

---

*Built with 🐕 by Mickey, Uncle John's robot dog*
