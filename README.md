# DiceRoll 3D 🎲

A modern 3D dice roller application built with Vue 3 and CSS 3D transforms.

## ✨ Features

| Feature | Description |
|---------|-------------|
| 🎲 **Dice Selection** | Roll 1-6 dice at once |
| 🎬 **3D Animation** | Realistic CSS 3D rolling animation |
| 📊 **Results** | Individual dice values + total sum |
| 📜 **History** | Session roll history with timestamps |
| 🌙 **Dark Mode** | Light/dark theme toggle |
| ✨ **Modern UI** | Clean, minimalist Tailwind design |

## 🚀 Getting Started

### Frontend

```bash
cd frontend
npm install
npm run dev
```

### Backend

```bash
cd backend
./mvnw spring-boot:run
```

## 🛠️ Tech Stack

- **Frontend**: Vue 3, TypeScript, Tailwind CSS, CSS 3D Transforms
- **Backend**: Spring Boot 3.3, Java 21

## 📖 How to Use

1. **Select dice count** - Click a number (1-6) to choose how many dice to roll
2. **Roll!** - Click the purple "¡Lanzar dados!" button
3. **Watch** - Enjoy the 3D rolling animation
4. **Results** - See individual values and total sum
5. **History** - Check the sidebar for past rolls
6. **Theme** - Toggle ☀️/🌙 for light/dark mode

## 📁 Project Structure

```
├── frontend/          # Vue 3 + Vite frontend
│   ├── src/
│   │   ├── views/     # Page components
│   │   ├── router/    # Vue Router config
│   │   └── assets/    # Static assets
│   └── package.json
├── backend/           # Spring Boot backend
│   ├── src/main/java/
│   └── pom.xml
└── README.md
```

## 📝 License

MIT
