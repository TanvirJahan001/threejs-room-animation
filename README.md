# 3D Developer Desk Animation

This project showcases a 3D developer desk setup created with Three.js and Vue.js. It features an animated keyboard typing effect, code appearing on the monitor screen, and a simulated compilation process.

## Features

- 3D rendered desk, chair, monitor, and keyboard
- Typing animation on the keyboard with text appearing on the monitor
- Code compilation simulation with changing status indicators
- Interactive 3D environment with orbit controls for camera movement

## Technologies Used

- Vue.js 3 (Composition API)
- Three.js for 3D rendering
- Vite for fast development and building

## Installation

1. Clone this repository
2. Install dependencies:
   ```
   npm install
   ```
3. Run the development server:
   ```
   npm run dev
   ```
4. Open your browser and navigate to `http://localhost:5173`

## How to Use

- Click and drag with your mouse to rotate the 3D view
- Scroll to zoom in and out
- Watch as code is typed on the keyboard and appears on the screen
- See the compilation process visually represented

## Project Structure

- `src/components/DeveloperDesk.vue` - The main Three.js component that renders the 3D scene
- `src/App.vue` - The main Vue application component

## License

MIT
