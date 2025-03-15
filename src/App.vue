<script setup>
import DeveloperDesk from './components/DeveloperDesk.vue'
</script>

<template>
  <div class="app-container">
    <header class="app-header">
      <h1>3D Developer Desk Animation</h1>
      <p>A Three.js demonstration with Vue.js showing a developer desk with typing and code compilation animation</p>
    </header>
    
    <main>
      <DeveloperDesk />
    </main>
    
    <footer class="app-footer">
      <p>Use mouse to drag and rotate the 3D scene. Scroll to zoom in/out.</p>
    </footer>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  background-color: #f8e1ff;
  overflow-x: hidden;
}

.app-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background: linear-gradient(135deg, #f8e1ff 0%, #e1c4ff 100%);
  position: relative;
}

.app-container::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: 
    radial-gradient(circle at 20% 30%, rgba(255, 158, 205, 0.15) 0%, transparent 50%),
    radial-gradient(circle at 80% 70%, rgba(225, 196, 255, 0.15) 0%, transparent 50%);
  pointer-events: none;
}

.app-header {
  padding: 2rem;
  text-align: center;
  background-color: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(10px);
  box-shadow: 
    0 4px 6px -1px rgba(0, 0, 0, 0.1),
    0 2px 4px -1px rgba(0, 0, 0, 0.06);
  position: relative;
  z-index: 10;
}

.app-header h1 {
  font-size: 2.5rem;
  margin-bottom: 0.75rem;
  color: #6c2bd9;
  font-weight: 800;
  letter-spacing: -0.025em;
  text-shadow: 2px 2px 4px rgba(108, 43, 217, 0.1);
}

.app-header p {
  color: #666;
  max-width: 600px;
  margin: 0 auto;
  line-height: 1.6;
  font-size: 1.1rem;
}

main {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 2rem;
  position: relative;
}

.threejs-container {
  width: 100%;
  height: 80vh;
  position: relative;
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 
    0 20px 25px -5px rgba(0, 0, 0, 0.1),
    0 10px 10px -5px rgba(0, 0, 0, 0.04);
}

.scene-container {
  width: 100%;
  height: 100%;
  background: transparent;
  border-radius: 20px;
}

.app-footer {
  padding: 1.5rem;
  text-align: center;
  background-color: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(10px);
  border-top: 1px solid rgba(255, 255, 255, 0.2);
  position: relative;
  z-index: 10;
}

.app-footer p {
  color: #666;
  font-size: 1rem;
  line-height: 1.5;
  max-width: 500px;
  margin: 0 auto;
}

@media (max-width: 768px) {
  .app-header {
    padding: 1.5rem 1rem;
  }

  .app-header h1 {
    font-size: 2rem;
  }

  .app-header p {
    font-size: 1rem;
  }

  main {
    padding: 1rem;
  }

  .threejs-container {
    height: 60vh;
  }
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.app-header h1,
.app-header p {
  animation: fadeIn 0.6s ease-out forwards;
}

.app-header p {
  animation-delay: 0.2s;
}
</style>
