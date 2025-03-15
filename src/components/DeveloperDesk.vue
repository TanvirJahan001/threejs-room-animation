<template>
  <div class="threejs-container">
    <div ref="threeContainer" class="scene-container"></div>
  </div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

export default {
  name: 'DeveloperDesk',
  setup() {
    const threeContainer = ref(null);
    const outputCode = ref('');
    const isCompiling = ref(false);
    const showUI = ref(false);
    const currentUIMockupIndex = ref(0);
    
    // Three.js variables
    let scene, camera, renderer, controls;
    let desk, monitor, keyboard;
    let compilingSphere;
    let animationFrameId;
    let initialCameraPosition = { x: 0, y: 3, z: 5 };
    
    // Compiling animation variables
    let atoms = []; // Move atoms array to component-level scope
    
    // Sample code snippets that will "type" on the monitor
    const codeSnippets = [
      `const button = document.querySelector('.btn');
button.addEventListener('click', () => {
  document.body.classList.toggle('dark-mode');
});`,
      `function createComponent() {
  return {
    template: '<div class="card">Hello World</div>',
    data() {
      return { count: 0 }
    }
  }
}`,
      `const colors = ['red', 'green', 'blue'];
colors.forEach(color => {
  console.log(\`The color is \${color}\`);
});`
    ];
    
    // Current typing state
    let currentSnippet = '';
    let currentSnippetIndex = 0;
    let currentLetterIndex = 0;
    let typingSpeed = 200; // Increased typing delay for more visible animation
    let lastTypingTime = 0;
    let lastKeyPressTime = 0;
    let keyPressDuration = 150; // Increased key press duration
    let isKeyPressed = false;
    
    // Reset camera to initial position (will be called automatically)
    const resetCamera = () => {
      if (camera && controls) {
        camera.position.set(
          initialCameraPosition.x,
          initialCameraPosition.y,
          initialCameraPosition.z
        );
        controls.target.set(0, 1.2, -0.5);
        controls.update();
      }
    };
    
    // Initialize the 3D scene
    const initScene = () => {
      // Create scene
      scene = new THREE.Scene();
      scene.background = new THREE.Color(0xf8e1ff); // Light pink background
      
      // Set up camera
      camera = new THREE.PerspectiveCamera(
        45,
        threeContainer.value.clientWidth / threeContainer.value.clientHeight,
        0.1,
        1000
      );
      
      // Position camera for better view
      camera.position.set(
        initialCameraPosition.x,
        initialCameraPosition.y,
        initialCameraPosition.z
      );
      
      // Set up renderer with improved shadows
      renderer = new THREE.WebGLRenderer({ 
        antialias: true,
        alpha: true
      });
      renderer.setSize(threeContainer.value.clientWidth, threeContainer.value.clientHeight);
      renderer.shadowMap.enabled = true;
      renderer.shadowMap.type = THREE.PCFSoftShadowMap;
      renderer.physicallyCorrectLights = true;
      renderer.toneMapping = THREE.ACESFilmicToneMapping;
      renderer.toneMappingExposure = 1.5; // Increased exposure
      threeContainer.value.appendChild(renderer.domElement);
      
      // Add controls for camera movement
      controls = new OrbitControls(camera, renderer.domElement);
      controls.enableDamping = true;
      controls.dampingFactor = 0.05;
      controls.maxPolarAngle = Math.PI / 2;
      controls.minDistance = 3;
      controls.maxDistance = 10;
      controls.target.set(0, 1, 0);
      controls.update();
      
      // Enhanced lighting setup
      // Ambient light
      const ambientLight = new THREE.AmbientLight(0xffffff, 1.0); // Increased intensity and changed color
      scene.add(ambientLight);
      
      // Main directional light
      const mainLight = new THREE.DirectionalLight(0xffffff, 2.0); // Increased intensity
      mainLight.position.set(5, 5, 5);
      mainLight.castShadow = true;
      mainLight.shadow.mapSize.width = 2048;
      mainLight.shadow.mapSize.height = 2048;
      mainLight.shadow.camera.near = 0.5;
      mainLight.shadow.camera.far = 500;
      mainLight.shadow.normalBias = 0.02;
      scene.add(mainLight);

      // Add soft fill light
      const fillLight = new THREE.DirectionalLight(0xffffff, 1.0); // Increased intensity
      fillLight.position.set(-5, 3, -5);
      scene.add(fillLight);

      // Add rim light for depth
      const rimLight = new THREE.DirectionalLight(0xffffff, 0.8); // Increased intensity
      rimLight.position.set(0, 5, -5);
      scene.add(rimLight);

      // Create room walls
      createRoom();
      
      // Create objects in order
      createDesk();
      createMonitor();
      createKeyboard();
      createDecor();
      createExtraDecor();
      createReactLikeCompilingAnimation();
      
      // Reset camera to initial position
      resetCamera();
      
      // Handle window resize
      window.addEventListener('resize', onWindowResize);
      
      // Start animation loop
      animate();
    };
    
    // Create room walls and floor
    const createRoom = () => {
      // Floor
      const floorGeometry = new THREE.PlaneGeometry(6, 6);
      const floorMaterial = new THREE.MeshStandardMaterial({ 
        color: 0xe1c4ff,
        roughness: 0.5,
        metalness: 0.1
      });
      const floor = new THREE.Mesh(floorGeometry, floorMaterial);
      floor.rotation.x = -Math.PI / 2;
      floor.receiveShadow = true;
      scene.add(floor);

      // Walls
      const wallMaterial = new THREE.MeshStandardMaterial({ 
        color: 0xb794f6,
        roughness: 0.5,
        metalness: 0.1
      });

      // Back wall
      const backWallGeometry = new THREE.PlaneGeometry(6, 4);
      const backWall = new THREE.Mesh(backWallGeometry, wallMaterial);
      backWall.position.z = -3;
      backWall.position.y = 2;
      backWall.receiveShadow = true;
      scene.add(backWall);

      // Side wall
      const sideWallGeometry = new THREE.PlaneGeometry(6, 4);
      const sideWall = new THREE.Mesh(sideWallGeometry, wallMaterial);
      sideWall.position.x = -3;
      sideWall.position.y = 2;
      sideWall.rotation.y = Math.PI / 2;
      sideWall.receiveShadow = true;
      scene.add(sideWall);

      // Add circular rug
      const rugGeometry = new THREE.CircleGeometry(1.5, 32);
      const rugMaterial = new THREE.MeshStandardMaterial({ 
        color: 0xff9ecd,
        roughness: 0.5,
        metalness: 0.1
      });
      const rug = new THREE.Mesh(rugGeometry, rugMaterial);
      rug.rotation.x = -Math.PI / 2;
      rug.position.y = 0.01;
      rug.receiveShadow = true;
      scene.add(rug);
    };

    // Create a simple desk - adjusted position
    const createDesk = () => {
      // Desk top
      const deskGeometry = new THREE.BoxGeometry(2.5, 0.05, 1.2);
      const deskMaterial = new THREE.MeshStandardMaterial({ 
        color: 0xffffff,
        roughness: 0.3,
        metalness: 0.2
      });
      desk = new THREE.Mesh(deskGeometry, deskMaterial);
      desk.position.y = 0.7;
      desk.position.z = -0.5;
      desk.receiveShadow = true;
      desk.castShadow = true;
      scene.add(desk);
      
      // Modern desk legs - single central support
      const legGeometry = new THREE.BoxGeometry(0.6, 0.68, 0.6);
      const legMaterial = new THREE.MeshStandardMaterial({ 
        color: 0xe1c4ff,
        roughness: 0.3,
        metalness: 0.2
      });
      
      const centralLeg = new THREE.Mesh(legGeometry, legMaterial);
      centralLeg.position.set(0, 0.35, -0.5);
      centralLeg.castShadow = true;
      scene.add(centralLeg);
      
      // Base plate
      const basePlateGeometry = new THREE.BoxGeometry(1, 0.02, 0.8);
      const basePlateMaterial = new THREE.MeshStandardMaterial({ 
        color: 0xb794f6,
        roughness: 0.3,
        metalness: 0.2
      });
      const basePlate = new THREE.Mesh(basePlateGeometry, basePlateMaterial);
      basePlate.position.set(0, 0.01, -0.5);
      basePlate.castShadow = true;
      scene.add(basePlate);
    };

    // Create decorative elements
    const createDecor = () => {
      // Vinyl records on wall with frames
      const createVinyl = (x, z) => {
        // Create frame
        const frameGeometry = new THREE.BoxGeometry(0.5, 0.5, 0.02);
        const frameMaterial = new THREE.MeshStandardMaterial({ 
          color: 0xffffff,
          roughness: 0.2
        });
        const frame = new THREE.Mesh(frameGeometry, frameMaterial);
        frame.position.set(x, 2.5, z);
        frame.rotation.y = Math.PI / 2;
        frame.castShadow = true;
        scene.add(frame);

        // Vinyl record
        const vinylGeometry = new THREE.CircleGeometry(0.2, 32);
        const vinylMaterial = new THREE.MeshStandardMaterial({ 
          color: 0x000000,
          roughness: 0.5
        });
        const vinyl = new THREE.Mesh(vinylGeometry, vinylMaterial);
        vinyl.position.set(x - 0.01, 2.5, z); // Slightly in front of frame
        vinyl.rotation.y = Math.PI / 2;
        vinyl.castShadow = true;
        scene.add(vinyl);
      };

      // Add multiple vinyl records
      createVinyl(-2.9, -2);
      createVinyl(-2.9, -1.5);
      createVinyl(-2.9, -1);

      // Add plant on desk
      const potGeometry = new THREE.CylinderGeometry(0.12, 0.08, 0.15);
      const potMaterial = new THREE.MeshStandardMaterial({ 
        color: 0xff9ecd,
        roughness: 0.8
      });
      const pot = new THREE.Mesh(potGeometry, potMaterial);
      pot.position.set(-0.8, 0.775, -0.7); // Positioned on desk
      pot.castShadow = true;
      scene.add(pot);

      // Enhanced plant leaves with better shape
      const createLeaf = (x, y, z, scale) => {
        const leafShape = new THREE.Shape();
        leafShape.moveTo(0, 0);
        leafShape.quadraticCurveTo(0.1, 0.2, 0, 0.4);
        leafShape.quadraticCurveTo(-0.1, 0.2, 0, 0);

        const leafGeometry = new THREE.ShapeGeometry(leafShape);
        const leafMaterial = new THREE.MeshStandardMaterial({ 
          color: 0x4CAF50,
          roughness: 0.8,
          side: THREE.DoubleSide
        });
        const leaf = new THREE.Mesh(leafGeometry, leafMaterial);
        leaf.position.set(x, y, z);
        leaf.scale.set(scale, scale, scale);
        leaf.rotation.set(
          Math.random() * Math.PI * 2,
          Math.random() * Math.PI * 2,
          Math.random() * Math.PI * 2
        );
        leaf.castShadow = true;
        scene.add(leaf);
      };

      // Create multiple leaves
      for (let i = 0; i < 12; i++) {
        createLeaf(
          -0.8 + Math.random() * 0.1 - 0.05,
          0.85 + Math.random() * 0.2,
          -0.7 + Math.random() * 0.1 - 0.05,
          0.15 + Math.random() * 0.1
        );
      }

      // Add picture frames on walls
      const createPictureFrame = (x, y, z, rotationY, width = 0.4, height = 0.3) => {
        const frameGeometry = new THREE.BoxGeometry(width, height, 0.02);
        const frameMaterial = new THREE.MeshStandardMaterial({ 
          color: 0xffffff,
          roughness: 0.2
        });
        const frame = new THREE.Mesh(frameGeometry, frameMaterial);
        frame.position.set(x, y, z);
        frame.rotation.y = rotationY;
        frame.castShadow = true;
        scene.add(frame);

        // Add picture content
        const pictureGeometry = new THREE.PlaneGeometry(width - 0.02, height - 0.02);
        const pictureMaterial = new THREE.MeshStandardMaterial({ 
          color: Math.random() > 0.5 ? 0xe1c4ff : 0xff9ecd,
          roughness: 0.5
        });
        const picture = new THREE.Mesh(pictureGeometry, pictureMaterial);
        picture.position.set(x + (rotationY === 0 ? 0 : 0.011), y, z + (rotationY === 0 ? 0.011 : 0));
        picture.rotation.y = rotationY;
        scene.add(picture);
      };

      // Add frames to both walls
      createPictureFrame(-2.99, 1.8, -1.5, Math.PI / 2, 0.5, 0.4);
      createPictureFrame(-2.99, 1.8, -0.5, Math.PI / 2, 0.3, 0.4);
      createPictureFrame(0, 1.8, -2.99, 0, 0.4, 0.3);
      createPictureFrame(-1, 1.8, -2.99, 0, 0.3, 0.4);

      // Add desk accessories
      // Mouse pad
      const mousePadGeometry = new THREE.BoxGeometry(0.25, 0.01, 0.3);
      const mousePadMaterial = new THREE.MeshStandardMaterial({ 
        color: 0xe1c4ff,
        roughness: 0.3
      });
      const mousePad = new THREE.Mesh(mousePadGeometry, mousePadMaterial);
      mousePad.position.set(0.5, 0.73, -0.3);
      mousePad.castShadow = true;
      scene.add(mousePad);

      // Mouse
      const mouseGeometry = new THREE.BoxGeometry(0.08, 0.03, 0.12);
      const mouseMaterial = new THREE.MeshStandardMaterial({ 
        color: 0xffffff,
        roughness: 0.2
      });
      const mouse = new THREE.Mesh(mouseGeometry, mouseMaterial);
      mouse.position.set(0.5, 0.75, -0.3);
      mouse.castShadow = true;
      scene.add(mouse);

      // Coffee mug
      const mugGeometry = new THREE.CylinderGeometry(0.04, 0.03, 0.08, 16);
      const mugMaterial = new THREE.MeshStandardMaterial({ 
        color: 0xff9ecd,
        roughness: 0.2
      });
      const mug = new THREE.Mesh(mugGeometry, mugMaterial);
      mug.position.set(0.8, 0.74, -0.6);
      mug.castShadow = true;
      scene.add(mug);
    };
    
    // Function to update the monitor display with code
    const updateMonitorDisplay = (code) => {
      if (!monitor || !monitor.displayCanvas) return;
      
      const canvas = monitor.displayCanvas;
      const ctx = canvas.getContext('2d');
      
      // Clear the canvas with VS Code dark theme background
      ctx.fillStyle = '#1E1E1E';
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      
      // Set text properties with larger font
      ctx.font = '28px "Courier New", monospace'; // Increased font size
      ctx.fillStyle = '#E4E4E4'; // Brighter text color
      
      // Split the code into lines
      const lines = code.split('\n');
      const lineHeight = 36; // Increased line height
      const padding = 40; // Increased padding
      
      // Draw editor gutter background
      ctx.fillStyle = '#252526';
      ctx.fillRect(0, 0, padding + 50, canvas.height); // Wider gutter
      
      // Draw line numbers and code
      lines.forEach((line, index) => {
        // Line number in gray
        ctx.fillStyle = '#858585';
        ctx.font = '24px "Courier New", monospace'; // Slightly smaller for line numbers
        ctx.fillText((index + 1).toString(), padding - 10, padding + index * lineHeight);
        
        // Enhanced syntax highlighting with brighter colors
        const tokens = tokenizeLine(line);
        let xPos = padding + 60; // Increased indent
        
        tokens.forEach(token => {
          ctx.font = '28px "Courier New", monospace'; // Reset font for code
          switch(token.type) {
            case 'string':
              ctx.fillStyle = '#FFA07A'; // Brighter orange for strings
              break;
            case 'keyword':
              ctx.fillStyle = '#569CD6'; // Bright blue for keywords
              break;
            case 'function':
              ctx.fillStyle = '#DCDCAA'; // Bright yellow for functions
              break;
            case 'number':
              ctx.fillStyle = '#B5CEA8'; // Bright green for numbers
              break;
            case 'comment':
              ctx.fillStyle = '#6A9955'; // Bright green for comments
              break;
            case 'operator':
              ctx.fillStyle = '#D4D4D4'; // White for operators
              break;
            default:
              ctx.fillStyle = '#E4E4E4'; // Bright white for default text
          }
          ctx.fillText(token.text, xPos, padding + index * lineHeight);
          xPos += ctx.measureText(token.text).width;
        });
      });
      
      // Add cursor with increased visibility
      const cursorBlinkRate = 530;
      if (Date.now() % (cursorBlinkRate * 2) < cursorBlinkRate) {
        const lastLine = lines.length - 1;
        const lastLineContent = lines[lastLine] || '';
        const cursorX = padding + 60 + ctx.measureText(lastLineContent).width;
        const cursorY = padding + lastLine * lineHeight;
        
        ctx.fillStyle = '#FFFFFF';
        ctx.fillRect(cursorX, cursorY - 24, 3, 32); // Thicker, taller cursor
      }
      
      // Update the texture
      monitor.displayTexture.needsUpdate = true;
    };
    
    // Helper function to tokenize code for syntax highlighting
    const tokenizeLine = (line) => {
      const tokens = [];
      const keywords = ['const', 'let', 'var', 'function', 'return', 'if', 'else', 'for', 'while'];
      const operators = ['+', '-', '*', '/', '=', '==', '===', '=>', '{', '}', '(', ')', '[', ']'];
      
      let current = '';
      let inString = false;
      let stringChar = '';
      
      for (let i = 0; i < line.length; i++) {
        const char = line[i];
        
        if (char === '"' || char === "'" || char === '`') {
          if (current) {
            tokens.push({ text: current, type: 'normal' });
            current = '';
          }
          if (inString && char === stringChar) {
            tokens.push({ text: current + char, type: 'string' });
            current = '';
            inString = false;
          } else if (!inString) {
            inString = true;
            stringChar = char;
            current = char;
          } else {
            current += char;
          }
          continue;
        }
        
        if (inString) {
          current += char;
          continue;
        }
        
        if (char === '/' && line[i + 1] === '/') {
          if (current) {
            tokens.push({ text: current, type: 'normal' });
          }
          tokens.push({ text: line.slice(i), type: 'comment' });
          break;
        }
        
        if (operators.includes(char)) {
          if (current) {
            tokens.push({ text: current, type: keywords.includes(current) ? 'keyword' : 'normal' });
            current = '';
          }
          tokens.push({ text: char, type: 'operator' });
          continue;
        }
        
        if (char === ' ') {
          if (current) {
            tokens.push({ text: current, type: keywords.includes(current) ? 'keyword' : 'normal' });
            current = '';
          }
          tokens.push({ text: ' ', type: 'normal' });
          continue;
        }
        
        current += char;
      }
      
      if (current) {
        tokens.push({ text: current, type: keywords.includes(current) ? 'keyword' : 'normal' });
      }
      
      return tokens;
    };
    
    // Function to show compiling animation on the monitor
    const showCompilingOnMonitor = (time) => {
      if (!monitor || !monitor.displayCanvas) return;
      
      const canvas = monitor.displayCanvas;
      const ctx = canvas.getContext('2d');
      
      // Clear the canvas with VS Code dark theme background
      ctx.fillStyle = '#1E1E1E';
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      
      const centerX = canvas.width / 2;
      const centerY = canvas.height / 2 - 20; // Move up slightly to make room for progress bar
      
      // Draw "Compiling..." text with pulsing effect
      const pulseValue = (Math.sin(time / 500) + 1) / 2; // Value between 0 and 1
      ctx.font = 'bold 24px "Courier New", monospace';
      ctx.fillStyle = `rgba(255, 255, 255, ${0.7 + pulseValue * 0.3})`;
      const compilingText = 'Compiling...';
      const textWidth = ctx.measureText(compilingText).width;
      ctx.fillText(compilingText, centerX - textWidth / 2, 50);
      
      // Draw React-like logo on the monitor with enhanced rotation
      const logoSize = 120; // Larger logo
      const rotationSpeed = time / 400; // Rotation based on time - faster for more obvious rotation
      
      // Add visible rotating circle around the React logo
      const outerCircleRadius = logoSize * 0.8;
      ctx.strokeStyle = '#4F46E5'; // Indigo color
      ctx.lineWidth = 4;
      
      // Draw rotating outer circle with dashes
      ctx.save();
      ctx.translate(centerX, centerY);
      ctx.rotate(rotationSpeed * 1.5); // Rotate in the opposite direction of the atoms
      
      // Create dashed circle
      ctx.beginPath();
      ctx.setLineDash([10, 15]);
      ctx.arc(0, 0, outerCircleRadius, 0, Math.PI * 2);
      ctx.stroke();
      
      // Add small dots along the rotating circle
      for (let i = 0; i < 8; i++) {
        const dotAngle = (i / 8) * Math.PI * 2;
        const dotX = outerCircleRadius * Math.cos(dotAngle);
        const dotY = outerCircleRadius * Math.sin(dotAngle);
        
        // Vary dot size based on position and time
        const dotPulse = (Math.sin(time / 300 + i) + 1) / 2;
        const dotSize = 4 + dotPulse * 3;
        
        ctx.fillStyle = '#61DAFB';
        ctx.beginPath();
        ctx.arc(dotX, dotY, dotSize, 0, Math.PI * 2);
        ctx.fill();
      }
      ctx.restore();
      
      // Add glow effect to the entire logo area
      const gradient = ctx.createRadialGradient(
        centerX, centerY, 0,
        centerX, centerY, logoSize
      );
      gradient.addColorStop(0, 'rgba(97, 218, 251, 0.2)');
      gradient.addColorStop(1, 'rgba(97, 218, 251, 0)');
      
      ctx.fillStyle = gradient;
      ctx.beginPath();
      ctx.arc(centerX, centerY, logoSize, 0, Math.PI * 2);
      ctx.fill();
      
      // Add spinning effect to the central circle
      ctx.save();
      ctx.translate(centerX, centerY);
      ctx.rotate(rotationSpeed);
      
      // Draw central circle with glow effect
      const glowSize = 8 + pulseValue * 5;
      ctx.shadowColor = '#61DAFB';
      ctx.shadowBlur = glowSize;
      ctx.shadowOffsetX = 0;
      ctx.shadowOffsetY = 0;
      
      ctx.fillStyle = '#61DAFB'; // React blue
      ctx.beginPath();
      
      // Draw central circle with spikes for more visible rotation
      const innerRadius = logoSize / 8;
      const outerRadius = logoSize / 6;
      const spikes = 6;
      
      for (let i = 0; i < spikes * 2; i++) {
        const angle = (i / (spikes * 2)) * Math.PI * 2;
        const radius = i % 2 === 0 ? outerRadius : innerRadius;
        const x = radius * Math.cos(angle);
        const y = radius * Math.sin(angle);
        
        if (i === 0) {
          ctx.moveTo(x, y);
        } else {
          ctx.lineTo(x, y);
        }
      }
      
      ctx.closePath();
      ctx.fill();
      ctx.restore();
      
      // Reset shadow for other elements
      ctx.shadowBlur = 0;
      
      // Draw orbiting ellipses
      const drawEllipse = (rotation) => {
        ctx.strokeStyle = '#61DAFB';
        ctx.lineWidth = 3;
        
        ctx.save();
        ctx.translate(centerX, centerY);
        ctx.rotate(rotation);
        ctx.scale(1, 0.4); // Make it elliptical
        
        ctx.beginPath();
        ctx.arc(0, 0, logoSize / 2, 0, Math.PI * 2);
        ctx.restore();
        ctx.stroke();
      };
      
      // Draw three ellipses at different rotations
      drawEllipse(rotationSpeed);
      drawEllipse(rotationSpeed + Math.PI / 3);
      drawEllipse(rotationSpeed + Math.PI * 2 / 3);
      
      // Draw orbiting atoms with glow
      const drawAtom = (angle, ellipseIndex) => {
        const ellipseRotation = rotationSpeed + (Math.PI / 3) * ellipseIndex;
        const x = centerX + Math.cos(angle) * (logoSize / 2) * Math.cos(ellipseRotation);
        const y = centerY + Math.sin(angle) * (logoSize / 2) * 0.4;
        
        ctx.shadowColor = '#61DAFB';
        ctx.shadowBlur = glowSize;
        
        ctx.fillStyle = '#61DAFB';
        ctx.beginPath();
        ctx.arc(x, y, 6, 0, Math.PI * 2);
        ctx.fill();
        
        ctx.shadowBlur = 0;
      };
      
      // Draw atoms on each ellipse - make more visible
      for (let i = 0; i < 3; i++) {
        drawAtom(rotationSpeed * 2 + i * Math.PI / 1.5, i);
        // Add a second atom to each ellipse for more motion
        drawAtom(rotationSpeed * 2 + i * Math.PI / 1.5 + Math.PI, i);
      }
      
      // Draw progress bar
      const progressBarWidth = 300;
      const progressBarHeight = 10;
      const progressBarX = centerX - progressBarWidth / 2;
      const progressBarY = centerY + 100;
      
      // Background of progress bar
      ctx.fillStyle = '#333333';
      ctx.fillRect(progressBarX, progressBarY, progressBarWidth, progressBarHeight);
      
      // Progress (pulsing effect)
      const progress = (time % 3000) / 3000; // Cycles every 3 seconds
      
      // Gradient for progress bar
      const progressGradient = ctx.createLinearGradient(progressBarX, 0, progressBarX + progressBarWidth * progress, 0);
      progressGradient.addColorStop(0, '#61DAFB');
      progressGradient.addColorStop(1, '#4F46E5');
      
      ctx.fillStyle = progressGradient;
      ctx.fillRect(progressBarX, progressBarY, progressBarWidth * progress, progressBarHeight);
      
      // Add a subtle glow to the progress bar
      ctx.shadowColor = '#61DAFB';
      ctx.shadowBlur = 10;
      ctx.fillRect(progressBarX, progressBarY, progressBarWidth * progress, progressBarHeight);
      ctx.shadowBlur = 0;
      
      // Add status text below progress bar
      ctx.font = '14px "Courier New", monospace';
      ctx.fillStyle = '#AAAAAA';
      
      // Cycle through different status messages
      const statusMessages = [
        'Optimizing code...',
        'Bundling modules...',
        'Transforming JSX...',
        'Applying hot reload...'
      ];
      const messageIndex = Math.floor((time / 1000) % statusMessages.length);
      const statusText = statusMessages[messageIndex];
      const statusWidth = ctx.measureText(statusText).width;
      ctx.fillText(statusText, centerX - statusWidth / 2, progressBarY + 30);
      
      // Update the texture
      monitor.displayTexture.needsUpdate = true;
    };
    
    // Function to show UI preview on the monitor
    const showUIPreviewOnMonitor = () => {
      if (!monitor || !monitor.displayCanvas) return;
      
      const canvas = monitor.displayCanvas;
      const ctx = canvas.getContext('2d');
      
      // Clear the canvas
      ctx.fillStyle = '#FFFFFF';
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      
      // Draw different UI based on current snippet index
      switch (currentUIMockupIndex.value) {
        case 0: // Dark mode toggle UI
          // Draw header
          ctx.fillStyle = '#F8FAFC';
          ctx.fillRect(0, 0, canvas.width, canvas.height);
          
          ctx.fillStyle = '#0F172A';
          ctx.font = 'bold 16px Arial, sans-serif';
          ctx.fillText('Theme Toggler', 20, 30);
          
          // Draw toggle button
          const toggleBtnX = canvas.width - 150;
          const toggleBtnY = 15;
          ctx.fillStyle = '#4F46E5';
          ctx.fillRect(toggleBtnX, toggleBtnY, 130, 30);
          
          ctx.fillStyle = '#FFFFFF';
          ctx.font = '14px Arial, sans-serif';
          ctx.fillText('Toggle Dark Mode', toggleBtnX + 10, toggleBtnY + 20);
          
          // Draw content
          ctx.fillStyle = '#0F172A';
          ctx.font = '14px Arial, sans-serif';
          ctx.fillText('This is a paragraph with some text content.', 20, 70);
          
          // Draw card
          ctx.fillStyle = '#FFFFFF';
          ctx.fillRect(20, 90, canvas.width - 40, 80);
          ctx.strokeStyle = '#E2E8F0';
          ctx.lineWidth = 1;
          ctx.strokeRect(20, 90, canvas.width - 40, 80);
          
          ctx.fillStyle = '#0F172A';
          ctx.font = 'bold 14px Arial, sans-serif';
          ctx.fillText('Card Title', 30, 110);
          
          ctx.font = '14px Arial, sans-serif';
          ctx.fillText('Card content that displays information.', 30, 130);
          
          // Draw button
          ctx.fillStyle = '#3B82F6';
          ctx.fillRect(30, 150, 80, 30);
          
          ctx.fillStyle = '#FFFFFF';
          ctx.fillText('Click me', 45, 170);
          break;
          
        case 1: // Component card UI
          // Draw card
          ctx.fillStyle = '#F5F5F5';
          ctx.fillRect(0, 0, canvas.width, canvas.height);
          
          const cardWidth = 300;
          const cardX = (canvas.width - cardWidth) / 2;
          
          // Card container
          ctx.fillStyle = '#FFFFFF';
          ctx.fillRect(cardX, 50, cardWidth, 200);
          ctx.shadowColor = 'rgba(0, 0, 0, 0.1)';
          ctx.shadowBlur = 10;
          ctx.shadowOffsetX = 0;
          ctx.shadowOffsetY = 4;
          
          // Card header
          ctx.fillStyle = '#4F46E5';
          ctx.fillRect(cardX, 50, cardWidth, 40);
          
          ctx.fillStyle = '#FFFFFF';
          ctx.font = 'bold 16px Arial, sans-serif';
          ctx.fillText('Hello World', cardX + 15, 75);
          
          // Card body
          ctx.shadowColor = 'transparent';
          ctx.fillStyle = '#000000';
          ctx.font = '14px Arial, sans-serif';
          ctx.fillText('This is a reusable component created from your code.', cardX + 15, 110);
          
          // Counter
          ctx.fillStyle = '#E5E7EB';
          ctx.beginPath();
          ctx.arc(cardX + 100, 140, 15, 0, Math.PI * 2);
          ctx.fill();
          
          ctx.fillStyle = '#000000';
          ctx.font = 'bold 14px Arial, sans-serif';
          ctx.fillText('-', cardX + 96, 145);
          
          ctx.fillStyle = '#000000';
          ctx.font = 'bold 16px Arial, sans-serif';
          ctx.fillText('0', cardX + 150, 145);
          
          ctx.fillStyle = '#E5E7EB';
          ctx.beginPath();
          ctx.arc(cardX + 200, 140, 15, 0, Math.PI * 2);
          ctx.fill();
          
          ctx.fillStyle = '#000000';
          ctx.font = 'bold 14px Arial, sans-serif';
          ctx.fillText('+', cardX + 196, 145);
          
          // Card footer
          ctx.fillStyle = '#F9FAFB';
          ctx.fillRect(cardX, 210, cardWidth, 40);
          
          ctx.fillStyle = '#4F46E5';
          ctx.fillRect(cardX + 200, 220, 80, 25);
          
          ctx.fillStyle = '#FFFFFF';
          ctx.font = '12px Arial, sans-serif';
          ctx.fillText('Learn More', cardX + 210, 235);
          break;
          
        case 2: // Color list UI
          // Draw header
          ctx.fillStyle = '#FFFFFF';
          ctx.fillRect(0, 0, canvas.width, canvas.height);
          
          ctx.fillStyle = '#111827';
          ctx.font = 'bold 16px Arial, sans-serif';
          ctx.fillText('Color List', 20, 30);
          
          // Draw color items
          const colors = [
            { name: 'red', hex: '#EF4444' },
            { name: 'green', hex: '#10B981' },
            { name: 'blue', hex: '#3B82F6' }
          ];
          
          colors.forEach((color, index) => {
            const y = 60 + index * 50;
            
            // Item container
            ctx.fillStyle = '#FFFFFF';
            ctx.fillRect(20, y, canvas.width - 40, 40);
            ctx.shadowColor = 'rgba(0, 0, 0, 0.05)';
            ctx.shadowBlur = 5;
            ctx.shadowOffsetX = 0;
            ctx.shadowOffsetY = 2;
            ctx.strokeStyle = '#F3F4F6';
            ctx.lineWidth = 1;
            ctx.strokeRect(20, y, canvas.width - 40, 40);
            
            // Color box
            ctx.shadowColor = 'transparent';
            ctx.fillStyle = color.hex;
            ctx.fillRect(30, y + 5, 30, 30);
            
            // Color label
            ctx.fillStyle = '#111827';
            ctx.font = '14px Arial, sans-serif';
            ctx.fillText(`The color is ${color.name}`, 70, y + 25);
          });
          break;
      }
      
      // Update the texture
      monitor.displayTexture.needsUpdate = true;
    };
    
    // Create a modern keyboard with individual keys
    const createKeyboard = () => {
      // Main keyboard body - larger and more visible
      const keyboardGeometry = new THREE.BoxGeometry(1.6, 0.1, 0.6);
      const keyboardMaterial = new THREE.MeshStandardMaterial({ 
        color: 0xe1c4ff, // Match room color theme
        roughness: 0.3,
        metalness: 0.4
      });
      keyboard = new THREE.Mesh(keyboardGeometry, keyboardMaterial);
      keyboard.position.set(0, 0.73, -0.3); // Moved forward for better visibility
      keyboard.rotation.x = 0.15; // Increased tilt
      keyboard.castShadow = true;
      scene.add(keyboard);

      // Create individual keys with improved visibility
      const keys = [];
      const keySize = 0.06; // Larger keys
      const keySpacing = 0.07; // More space between keys
      const keyHeight = 0.02; // Taller keys
      const keyGeometry = new THREE.BoxGeometry(keySize, keyHeight, keySize);
      const keyMaterial = new THREE.MeshStandardMaterial({
        color: 0xffffff,
        roughness: 0.3,
        metalness: 0.3,
        emissive: 0x666666,
        emissiveIntensity: 0.1
      });
      const keyActiveMaterial = new THREE.MeshStandardMaterial({
        color: 0xff9ecd, // Match room accent color
        roughness: 0.2,
        metalness: 0.6,
        emissive: 0xff9ecd,
        emissiveIntensity: 0.5
      });

      // Define full keyboard layout with better spacing
      const keyboardLayout = [
        ['ESC', '1', '2', '3', '4', '5', '6', '7', '8', '9', '0', '-', '=', 'BACK'],
        ['TAB', 'Q', 'W', 'E', 'R', 'T', 'Y', 'U', 'I', 'O', 'P', '[', ']', '\\'],
        ['CAPS', 'A', 'S', 'D', 'F', 'G', 'H', 'J', 'K', 'L', ';', "'", 'ENTER'],
        ['SHIFT', 'Z', 'X', 'C', 'V', 'B', 'N', 'M', ',', '.', '/', 'SHIFT'],
        ['CTRL', 'FN', 'WIN', 'ALT', 'SPACE', 'ALT', 'CTRL', '←', '↑', '↓', '→']
      ];

      // Special key sizes (multiplier for standard key size)
      const specialKeySizes = {
        'ESC': 1.2,
        'BACK': 2,
        'TAB': 1.5,
        'CAPS': 1.75,
        'ENTER': 2.25,
        'SHIFT': 2.25,
        'CTRL': 1.25,
        'FN': 1,
        'WIN': 1.25,
        'ALT': 1.25,
        'SPACE': 6.25
      };

      // Create keys for each row with improved positioning
      keyboardLayout.forEach((row, rowIndex) => {
        let xOffset = -0.7; // Start further left
        
        row.forEach((keyLabel) => {
          // Calculate key width based on special sizes
          const keyWidth = (specialKeySizes[keyLabel] || 1) * keySize;
          const keyGeom = new THREE.BoxGeometry(keyWidth, keyHeight, keySize);
          
          // Create key cap with rounded edges
          const key = new THREE.Mesh(keyGeom, keyMaterial.clone());
          
          // Position the key
          const x = xOffset + (keyWidth / 2);
          const z = (rowIndex * keySpacing) - 0.25;
          const y = 0.78 + keyHeight / 2;
          key.position.set(x, y, z);
          key.rotation.x = 0.15; // Match keyboard tilt
          
          // Add key cap top with slight bevel
          const topGeometry = new THREE.BoxGeometry(keyWidth * 0.95, 0.002, keySize * 0.95);
          const topMesh = new THREE.Mesh(topGeometry, keyMaterial.clone());
          topMesh.position.y = keyHeight / 2;
          key.add(topMesh);
          
          // Add subtle shadow under keys
          const shadowGeometry = new THREE.PlaneGeometry(keyWidth, keySize);
          const shadowMaterial = new THREE.MeshBasicMaterial({
            color: 0x000000,
            transparent: true,
            opacity: 0.2
          });
          const shadow = new THREE.Mesh(shadowGeometry, shadowMaterial);
          shadow.rotation.x = -Math.PI / 2;
          shadow.position.y = -keyHeight / 2 - 0.001;
          key.add(shadow);
          
          // Store key data
          key.userData = {
            letter: keyLabel,
            originalY: y,
            material: key.material,
            activeMaterial: keyActiveMaterial.clone(),
            isPressed: false,
            pressStartTime: 0,
            x: x,
            z: z,
            width: keyWidth,
            topMesh: topMesh
          };
          
          scene.add(key);
          keys.push(key);
          
          // Update offset for next key
          xOffset += keyWidth + (keySpacing - keySize) * 0.8;
        });
      });

      // Store keys in keyboard object for animation
      keyboard.userData = { keys };
    };
    
    // Create React-like compiling animation
    const createReactLikeCompilingAnimation = () => {
      // Create a group for the React-like logo
      compilingSphere = new THREE.Group();
      compilingSphere.position.set(2, 1.3, -0.8);
      compilingSphere.scale.set(0.6, 0.6, 0.6);
      compilingSphere.visible = true;
      scene.add(compilingSphere);
      
      // Add spotlights to highlight the animation
      const spotLight1 = new THREE.SpotLight(0x4F46E5, 2, 5, Math.PI / 6, 0.5, 1);
      spotLight1.position.set(2, 2, 0);
      spotLight1.target = compilingSphere;
      scene.add(spotLight1);
      
      const spotLight2 = new THREE.SpotLight(0x61DAFB, 2, 5, Math.PI / 6, 0.5, 1);
      spotLight2.position.set(0, 2, 0);
      spotLight2.target = compilingSphere;
      scene.add(spotLight2);
      
      // Create the central circle with stronger glow
      const centerGeometry = new THREE.SphereGeometry(0.15, 32, 32);
      const centerMaterial = new THREE.MeshStandardMaterial({ 
        color: 0x61DAFB, // React blue
        emissive: 0x61DAFB,
        emissiveIntensity: 1.2, // Increase intensity
        transparent: true,
        opacity: 0.9
      });
      const centerSphere = new THREE.Mesh(centerGeometry, centerMaterial);
      compilingSphere.add(centerSphere);
      
      // Add a stronger glow effect to the central sphere
      const glowGeometry = new THREE.SphereGeometry(0.2, 32, 32);
      const glowMaterial = new THREE.MeshBasicMaterial({
        color: 0x61DAFB,
        transparent: true,
        opacity: 0.5 // Increase opacity
      });
      const glowSphere = new THREE.Mesh(glowGeometry, glowMaterial);
      compilingSphere.add(glowSphere);

      // Add outer glow for more visibility
      const outerGlowGeometry = new THREE.SphereGeometry(0.25, 32, 32);
      const outerGlowMaterial = new THREE.MeshBasicMaterial({
        color: 0x61DAFB,
        transparent: true,
        opacity: 0.2
      });
      const outerGlowSphere = new THREE.Mesh(outerGlowGeometry, outerGlowMaterial);
      compilingSphere.add(outerGlowSphere);
      
      // Create the orbiting ellipses with brighter materials
      const createEllipse = (rotation) => {
        const curve = new THREE.EllipseCurve(
          0, 0,            // Center x, y
          0.5, 0.5,        // xRadius, yRadius
          0, 2 * Math.PI,  // Start angle, end angle
          false,           // Clockwise
          0                // Rotation
        );
        
        const points = curve.getPoints(50);
        const geometry = new THREE.BufferGeometry().setFromPoints(points);
        
        const material = new THREE.LineBasicMaterial({ 
          color: 0x61DAFB,
          transparent: true,
          opacity: 0.9, // Increase opacity
          linewidth: 2 // Note: linewidth only works in some browsers
        });
        
        const ellipse = new THREE.Line(geometry, material);
        ellipse.rotation.set(rotation.x, rotation.y, rotation.z);
        return ellipse;
      };
      
      // Add three ellipses at different angles
      const ellipse1 = createEllipse({ x: 0, y: 0, z: 0 });
      const ellipse2 = createEllipse({ x: Math.PI / 3, y: Math.PI / 6, z: 0 });
      const ellipse3 = createEllipse({ x: Math.PI / 6, y: Math.PI / 3, z: Math.PI / 4 });
      
      compilingSphere.add(ellipse1);
      compilingSphere.add(ellipse2);
      compilingSphere.add(ellipse3);
      
      // Add small orbiting atoms with enhanced visibility
      const atomGeometry = new THREE.SphereGeometry(0.06, 16, 16); // Slightly larger
      const atomMaterial = new THREE.MeshStandardMaterial({ 
        color: 0x61DAFB,
        emissive: 0x61DAFB,
        emissiveIntensity: 1.0,
        transparent: true,
        opacity: 1.0 // Full opacity
      });
      
      // Create atoms for each ellipse - add more atoms for more movement
      atoms = [];
      
      // Create 2 atoms for each ellipse
      for (let i = 0; i < 3; i++) {
        for (let j = 0; j < 2; j++) {
          const atom = new THREE.Mesh(atomGeometry, atomMaterial);
          atom.userData = {
            ellipseIndex: i,
            angle: (j * Math.PI) + (Math.random() * Math.PI / 4), // Space them apart
            speed: 0.02 + Math.random() * 0.01
          };
          
          // Add glow to atoms
          const atomGlow = new THREE.PointLight(0x61DAFB, 0.5, 0.3);
          atom.add(atomGlow);
          
          compilingSphere.add(atom);
          atoms.push(atom);
        }
      }
      
      compilingSphere.userData = { 
        ellipses: [ellipse1, ellipse2, ellipse3],
        atoms: atoms,
        centerSphere,
        glowSphere,
        outerGlowSphere
      };
    };
    
    // Add more decorative elements to the room
    const createExtraDecor = () => {
      // Add floating shelves on the wall
      const createShelf = (x, y, z, width) => {
        const shelfGeometry = new THREE.BoxGeometry(width, 0.05, 0.2);
        const shelfMaterial = new THREE.MeshStandardMaterial({ 
          color: 0xffffff,
          roughness: 0.2
        });
        const shelf = new THREE.Mesh(shelfGeometry, shelfMaterial);
        shelf.position.set(x, y, z);
        shelf.castShadow = true;
        shelf.receiveShadow = true;
        scene.add(shelf);
        return shelf;
      };

      // Add shelves to both walls
      createShelf(-2.9, 1.2, -1.0, 0.8);
      createShelf(-2.9, 1.2, -2.0, 0.8);
      createShelf(-1.5, 1.2, -2.9, 0.8);

      // Add books on shelves
      const createBook = (x, y, z, color, width = 0.04) => {
        const bookGeometry = new THREE.BoxGeometry(width, 0.15, 0.15);
        const bookMaterial = new THREE.MeshStandardMaterial({ 
          color: color,
          roughness: 0.3
        });
        const book = new THREE.Mesh(bookGeometry, bookMaterial);
        book.position.set(x, y, z);
        book.castShadow = true;
        scene.add(book);
      };

      // Add groups of books
      const bookColors = [0xff9ecd, 0xe1c4ff, 0xb794f6, 0x4F46E5];
      for (let i = 0; i < 8; i++) {
        createBook(-2.9, 1.3, -1.0 + (i * 0.05), bookColors[i % bookColors.length]);
      }
      for (let i = 0; i < 6; i++) {
        createBook(-2.9, 1.3, -2.0 + (i * 0.05), bookColors[(i + 2) % bookColors.length]);
      }

      // Add a small clock on the desk
      const clockBase = new THREE.CylinderGeometry(0.06, 0.06, 0.02, 32);
      const clockMaterial = new THREE.MeshStandardMaterial({ 
        color: 0xffffff,
        roughness: 0.2
      });
      const clock = new THREE.Mesh(clockBase, clockMaterial);
      clock.position.set(-0.9, 0.73, -0.3);
      clock.castShadow = true;
      scene.add(clock);

      // Add some sticky notes
      const createStickyNote = (x, y, z, rotation, color) => {
        const noteGeometry = new THREE.BoxGeometry(0.1, 0.1, 0.001);
        const noteMaterial = new THREE.MeshStandardMaterial({ 
          color: color,
          roughness: 0.3
        });
        const note = new THREE.Mesh(noteGeometry, noteMaterial);
        note.position.set(x, y, z);
        note.rotation.set(...rotation);
        note.castShadow = true;
        scene.add(note);
      };

      // Add sticky notes in different positions
      createStickyNote(-0.6, 0.73, -0.4, [0, Math.PI / 6, 0], 0xff9ecd);
      createStickyNote(-0.65, 0.731, -0.4, [0, -Math.PI / 8, 0], 0xe1c4ff);
      createStickyNote(-0.62, 0.732, -0.35, [0, Math.PI / 4, 0], 0xb794f6);
    };
    
    // Animation loop
    const animate = (time) => {
      animationFrameId = requestAnimationFrame(animate);
      
      if (controls) controls.update();
      
      // Handle key press animation timing
      if (isKeyPressed && time - lastKeyPressTime > keyPressDuration) {
        // Reset keys position
        isKeyPressed = false;
        resetKeysPosition();
      }
      
      // Simulate typing on the keyboard
      if (time - lastTypingTime > typingSpeed) {
        lastTypingTime = time;
        
        if (currentLetterIndex < codeSnippets[currentSnippetIndex].length) {
          // Get the current character being typed
          const currentChar = codeSnippets[currentSnippetIndex][currentLetterIndex];
          
          // Still typing the current snippet
          currentSnippet += currentChar;
          currentLetterIndex++;
          outputCode.value = currentSnippet;
          
          // Update the monitor display with the current code
          updateMonitorDisplay(currentSnippet);
          
          // Trigger key press animation
          animateKeyPress(time, currentChar);
          
          // Set typing speed based on character type for more realistic typing
          if (currentChar === ' ' || currentChar === '\n') {
            typingSpeed = 300; // Longer pause at spaces and new lines
          } else if (currentChar === '.' || currentChar === ',' || currentChar === ';') {
            typingSpeed = 250; // Longer pause at punctuation
          } else if (currentChar === '{' || currentChar === '}') {
            typingSpeed = 350; // Even longer pause at braces
          } else {
            typingSpeed = 200 + Math.random() * 100; // Random variation in typing speed
          }
          
          // Occasionally add a longer pause to simulate thinking
          if (Math.random() < 0.1) {
            typingSpeed += 400;
          }
        } else if (time - lastTypingTime > 2000 && !isCompiling.value) {
          // Reset typing speed for next snippet
          typingSpeed = 100;
          
          // Finished typing, show "compiling" status
          isCompiling.value = true;
          showUI.value = false;
          
          // Show compiling animation on the monitor immediately
          showCompilingOnMonitor(time);
          
          // After a delay, show the UI mockup directly on the monitor
          setTimeout(() => {
            isCompiling.value = false;
            showUI.value = true;
            currentUIMockupIndex.value = currentSnippetIndex;
            
            // Show UI preview on the monitor
            showUIPreviewOnMonitor();
            
            // Reset for next snippet after viewing UI
            setTimeout(() => {
              showUI.value = false;
              currentSnippetIndex = (currentSnippetIndex + 1) % codeSnippets.length;
              currentLetterIndex = 0;
              currentSnippet = '';
              
              // Clear the monitor display for the next code snippet
              if (monitor && monitor.displayCanvas) {
                const ctx = monitor.displayCanvas.getContext('2d');
                ctx.fillStyle = '#1E1E1E';
                ctx.fillRect(0, 0, monitor.displayCanvas.width, monitor.displayCanvas.height);
                monitor.displayTexture.needsUpdate = true;
              }
            }, 4000);
          }, 3000);
        }
      }
      
      // Update the blinking cursor on the monitor
      if (currentSnippet && monitor && monitor.displayTexture && !isCompiling.value && !showUI.value) {
        if (time % 1000 < 10) { // Only update occasionally to avoid performance issues
          updateMonitorDisplay(currentSnippet);
        }
      }
      
      // Show compiling animation on the monitor when compiling
      if (isCompiling.value && monitor && monitor.displayTexture) {
        showCompilingOnMonitor(time);
      }
      
      // Always update the 3D compiling sphere animation regardless of compilation state
      updateCompilingSphereAnimation(time);
      
      renderer.render(scene, camera);
    };
    
    // Animate keyboard for a key press - enhanced for better realism
    const animateKeyPress = (time, character) => {
      isKeyPressed = true;
      lastKeyPressTime = time;
      
      if (keyboard && keyboard.userData && keyboard.userData.keys) {
        const keys = keyboard.userData.keys;
        
        // Reset keys that have finished their press animation
        keys.forEach(key => {
          if (key.userData.isPressed && (time - key.userData.pressStartTime > keyPressDuration)) {
            key.position.y = key.userData.originalY;
            key.material = key.userData.material;
            key.userData.isPressed = false;
          }
        });
        
        // Find the corresponding key for the character
        const charToFind = character.toUpperCase();
        let mainKey = keys.find(key => key.userData.letter === charToFind);
        
        // Handle special characters
        if (!mainKey) {
          switch (character) {
            case ' ':
              mainKey = keys.find(key => key.userData.letter === 'SPACE');
              break;
            case '\n':
              mainKey = keys.find(key => key.userData.letter === 'ENTER');
              break;
            case '{':
              mainKey = keys.find(key => key.userData.letter === '[');
              break;
            case '}':
              mainKey = keys.find(key => key.userData.letter === ']');
              break;
            // Add more special character mappings as needed
          }
        }
        
        // Animate the main key with enhanced visual feedback
        if (mainKey) {
          // Deeper press for more visible movement
          mainKey.position.y = mainKey.userData.originalY - 0.02; // Increased press depth
          mainKey.material = mainKey.userData.activeMaterial;
          mainKey.userData.isPressed = true;
          mainKey.userData.pressStartTime = time;
          
          // Enhanced glow effect
          mainKey.userData.activeMaterial.emissiveIntensity = 1.0; // Increased glow
          
          // Animate nearby keys with subtle movement
          const nearbyKeys = keys.filter(key => {
            if (key === mainKey || key.userData.isPressed) return false;
            const dx = key.userData.x - mainKey.userData.x;
            const dz = key.userData.z - mainKey.userData.z;
            const distance = Math.sqrt(dx * dx + dz * dz);
            return distance < 0.15;
          });
          
          nearbyKeys.forEach(key => {
            key.position.y = key.userData.originalY - 0.005;
            key.material.emissiveIntensity = 0.3;
            setTimeout(() => {
              if (!key.userData.isPressed) {
                key.position.y = key.userData.originalY;
                key.material.emissiveIntensity = 0.1;
              }
            }, 100);
          });
          
          // Animate modifier keys for special characters
          if (character.match(/[A-Z!@#$%^&*(){}]/)) {
            const shiftKeys = keys.filter(key => key.userData.letter === 'SHIFT');
            shiftKeys.forEach(key => {
              key.position.y = key.userData.originalY - 0.015;
              key.material = key.userData.activeMaterial;
              key.userData.isPressed = true;
              key.userData.pressStartTime = time;
            });
          }
        }
      }
    };
    
    // Reset all keyboard keys to their original position
    const resetKeysPosition = () => {
      if (keyboard && keyboard.userData && keyboard.userData.keys) {
        const keys = keyboard.userData.keys;
        
        keys.forEach(key => {
          if (key.userData.isPressed) {
            key.position.y = key.userData.originalY;
            key.material = key.userData.material;
            key.userData.isPressed = false;
          }
        });
      }
    };
    
    // Handle window resize
    const onWindowResize = () => {
      if (camera && renderer && threeContainer.value) {
        camera.aspect = threeContainer.value.clientWidth / threeContainer.value.clientHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(threeContainer.value.clientWidth, threeContainer.value.clientHeight);
      }
    };
    
    // Clean up
    const cleanup = () => {
      if (animationFrameId) {
        cancelAnimationFrame(animationFrameId);
      }
      
      if (renderer && threeContainer.value) {
        threeContainer.value.removeChild(renderer.domElement);
      }
      
      window.removeEventListener('resize', onWindowResize);
    };
    
    // Function to update the compiling sphere animation
    const updateCompilingSphereAnimation = (time) => {
      if (!compilingSphere) return;
      
      // Pulsing effect for the center and glow spheres
      const pulse = (Math.sin(time / 1000) + 1) / 2; // Value between 0 and 1
      
      // Apply pulsing to center and glow
      if (compilingSphere.children[0]) { // Center sphere
        compilingSphere.children[0].scale.set(
          1 + pulse * 0.1,
          1 + pulse * 0.1,
          1 + pulse * 0.1
        );
      }
      
      if (compilingSphere.children[1]) { // Inner glow sphere
        compilingSphere.children[1].scale.set(
          1 + pulse * 0.15,
          1 + pulse * 0.15,
          1 + pulse * 0.15
        );
      }
      
      if (compilingSphere.children[2]) { // Outer glow sphere
        compilingSphere.children[2].scale.set(
          1 + pulse * 0.2,
          1 + pulse * 0.2,
          1 + pulse * 0.2
        );
      }
      
      // Add a subtle floating movement to the entire compilingSphere
      compilingSphere.position.y = 1.5 + Math.sin(time / 1500) * 0.05;
      
      // Rotate the entire compilingSphere
      compilingSphere.rotation.y += 0.01;
      
      // Rotate each ellipse
      if (compilingSphere.children) {
        // The center sphere is at index 0, glows at index 1 and 2
        // Ellipses are at indices 3, 4, 5
        if (compilingSphere.children.length > 3) {
          // Rotate each ellipse at a different speed
          compilingSphere.children[3].rotation.z += 0.005;
          if (compilingSphere.children.length > 4) {
            compilingSphere.children[4].rotation.x += 0.007;
          }
          if (compilingSphere.children.length > 5) {
            compilingSphere.children[5].rotation.y += 0.009;
          }
        }
        
        // Update atom positions
        atoms.forEach((atom, index) => {
          const ellipseIndex = atom.userData.ellipseIndex;
          atom.userData.angle += atom.userData.speed;
          
          // Calculate position based on ellipse
          const radius = 0.5;
          const x = Math.cos(atom.userData.angle) * radius;
          const y = Math.sin(atom.userData.angle) * radius;
          
          // Set atom position based on its ellipse
          switch(ellipseIndex) {
            case 0:
              atom.position.set(x, y, 0);
              break;
            case 1:
              atom.position.set(x, 0, y);
              break;
            case 2:
              atom.position.set(0, x, y);
              break;
          }
          
          // Scale atoms with a different pulse for more interesting effect
          const atomPulse = (Math.sin(time / 800 + index) + 1) / 2;
          atom.scale.set(
            1 + atomPulse * 0.2,
            1 + atomPulse * 0.2,
            1 + atomPulse * 0.2
          );
        });
      }
    };
    
    // Create a modern monitor
    const createMonitor = () => {
      // Monitor screen (outer frame)
      const screenGeometry = new THREE.BoxGeometry(2.0, 1.1, 0.08);
      const screenMaterial = new THREE.MeshStandardMaterial({ 
        color: 0x333333,
        roughness: 0.2,
        metalness: 0.5
      });
      const screen = new THREE.Mesh(screenGeometry, screenMaterial);
      screen.position.set(0, 1.4, -1.2);
      screen.castShadow = true;
      scene.add(screen);
      
      // Create a canvas for the monitor display with higher resolution
      const displayCanvas = document.createElement('canvas');
      displayCanvas.width = 2560; // Increased resolution for sharper text
      displayCanvas.height = 1440; // Increased resolution for sharper text
      const ctx = displayCanvas.getContext('2d');
      
      // Fill with dark background (VS Code-like)
      ctx.fillStyle = '#1E1E1E';
      ctx.fillRect(0, 0, displayCanvas.width, displayCanvas.height);
      
      // Create texture from canvas with better filtering
      const displayTexture = new THREE.CanvasTexture(displayCanvas);
      displayTexture.minFilter = THREE.LinearFilter;
      displayTexture.magFilter = THREE.LinearFilter;
      displayTexture.anisotropy = 16; // Increase anisotropy for sharper text at angles
      
      // Monitor display (screen surface)
      const displayGeometry = new THREE.PlaneGeometry(1.92, 1.02); // Slightly smaller than frame
      const displayMaterial = new THREE.MeshBasicMaterial({ 
        map: displayTexture,
        side: THREE.FrontSide
      });
      const display = new THREE.Mesh(displayGeometry, displayMaterial);
      display.position.set(0, 1.4, -1.159); // Adjusted to be just in front of the screen
      scene.add(display);
      
      // Modern monitor stand - sleek design
      const standGeometry = new THREE.BoxGeometry(0.1, 0.4, 0.1);
      const standMaterial = new THREE.MeshStandardMaterial({ 
        color: 0x333333,
        roughness: 0.2,
        metalness: 0.5
      });
      const stand = new THREE.Mesh(standGeometry, standMaterial);
      stand.position.set(0, 1.0, -1.25);
      stand.castShadow = true;
      scene.add(stand);
      
      // Monitor base - modern oval shape
      const baseGeometry = new THREE.CylinderGeometry(0.25, 0.3, 0.02, 32);
      const base = new THREE.Mesh(baseGeometry, standMaterial);
      base.position.set(0, 0.75, -1.25);
      base.castShadow = true;
      scene.add(base);
      
      // Add thin bezel around display
      const bezelTop = new THREE.Mesh(
        new THREE.BoxGeometry(2.0, 0.04, 0.02),
        screenMaterial
      );
      bezelTop.position.set(0, 1.91, -1.16);
      scene.add(bezelTop);
      
      const bezelBottom = new THREE.Mesh(
        new THREE.BoxGeometry(2.0, 0.04, 0.02),
        screenMaterial
      );
      bezelBottom.position.set(0, 0.89, -1.16);
      scene.add(bezelBottom);
      
      const bezelLeft = new THREE.Mesh(
        new THREE.BoxGeometry(0.04, 1.1, 0.02),
        screenMaterial
      );
      bezelLeft.position.set(-0.98, 1.4, -1.16);
      scene.add(bezelLeft);
      
      const bezelRight = new THREE.Mesh(
        new THREE.BoxGeometry(0.04, 1.1, 0.02),
        screenMaterial
      );
      bezelRight.position.set(0.98, 1.4, -1.16);
      scene.add(bezelRight);
      
      monitor = { 
        screen, 
        display, 
        displayCanvas, 
        displayTexture,
        stand, 
        base 
      };
    };
    
    onMounted(() => {
      initScene();
    });
    
    onBeforeUnmount(() => {
      cleanup();
    });
    
    return {
      threeContainer,
      outputCode,
      isCompiling,
      showUI,
      currentUIMockupIndex
    };
  }
};
</script>

<style scoped>
.threejs-container {
  position: relative;
  width: 100%;
  height: 100%;
  min-height: 500px;
  overflow: hidden;
  background: linear-gradient(135deg, #f8e1ff 0%, #e1c4ff 100%);
}

.scene-container {
  width: 100%;
  height: 100%;
  min-height: 500px;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

@keyframes toggleMode {
  0%, 45% { background: #f8fafc; color: #0f172a; }
  55%, 100% { background: #0f172a; color: #f8fafc; }
}

@keyframes cardPulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.02); }
}
</style> 