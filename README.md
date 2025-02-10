import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';

// Title
document.title = 'GamingBoost';

// Scene, Camera, Renderer
const scene = new THREE.Scene();
scene.fog = new THREE.Fog(0xaaaaaa, 10, 50);
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer({ antialias: true, physicallyCorrectLights: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
document.body.appendChild(renderer.domElement);

// Add lighting
const light = new THREE.DirectionalLight(0xffffff, 1.5);
light.position.set(5, 10, 5);
light.castShadow = true;
scene.add(light);

const ambientLight = new THREE.AmbientLight(0x606060);
scene.add(ambientLight);

// Load realistic player model with animations
const loader = new GLTFLoader();
let player, mixer;
loader.load('realistic_player.glb', (gltf) => {
    player = gltf.scene;
    player.scale.set(1, 1, 1);
    player.traverse((child) => {
        if (child.isMesh) {
            child.castShadow = true;
            child.receiveShadow = true;
        }
    });
    scene.add(player);
    
    // Add animations
    mixer = new THREE.AnimationMixer(player);
    if (gltf.animations.length > 0) {
        const action = mixer.clipAction(gltf.animations[0]);
        action.play();
    }
});

// Position camera
camera.position.set(0, 3, 10);
camera.lookAt(0, 1, 0);

// Player movement
let moveDirection = { left: false, right: false, jump: false };
document.addEventListener('keydown', (event) => {
    if (event.key === 'ArrowLeft') moveDirection.left = true;
    if (event.key === 'ArrowRight') moveDirection.right = true;
    if (event.key === 'ArrowUp') moveDirection.jump = true;
});

document.addEventListener('keyup', (event) => {
    if (event.key === 'ArrowLeft') moveDirection.left = false;
    if (event.key === 'ArrowRight') moveDirection.right = false;
    if (event.key === 'ArrowUp') moveDirection.jump = false;
});

// Mobile controls
let touchStartX = 0;
document.addEventListener('touchstart', (event) => {
    touchStartX = event.touches[0].clientX;
});

document.addEventListener('touchmove', (event) => {
    let touchEndX = event.touches[0].clientX;
    let diffX = touchEndX - touchStartX;
    if (diffX > 50) moveDirection.right = true;
    if (diffX < -50) moveDirection.left = true;
});

document.addEventListener('touchend', () => {
    moveDirection.left = false;
    moveDirection.right = false;
});

document.addEventListener('touchstart', (event) => {
    if (event.touches.length > 1) moveDirection.jump = true;
});

document.addEventListener('touchend', () => {
    moveDirection.jump = false;
});

// Score System
let score = 0;
const scoreElement = document.createElement('div');
scoreElement.style.position = 'absolute';
scoreElement.style.top = '10px';
scoreElement.style.left = '10px';
scoreElement.style.color = 'white';
scoreElement.style.fontSize = '20px';
document.body.appendChild(scoreElement);

// Infinite Ground Generation
const groundGeometry = new THREE.PlaneGeometry(10, 50);
const groundMaterial = new THREE.MeshStandardMaterial({ color: 0x228B22 });
const ground = new THREE.Mesh(groundGeometry, groundMaterial);
ground.rotation.x = -Math.PI / 2;
ground.position.y = 0;
ground.receiveShadow = true;
scene.add(ground);

function updateGround() {
    ground.position.z -= 0.1;
    if (ground.position.z < -25) {
        ground.position.z = 0;
    }
}

// Animation loop
const clock = new THREE.Clock();
function animate() {
    requestAnimationFrame(animate);
    const delta = clock.getDelta();
    if (mixer) mixer.update(delta);
    
    if (player) {
        if (moveDirection.left) player.position.x -= 0.1;
        if (moveDirection.right) player.position.x += 0.1;
        if (moveDirection.jump) player.position.y += 0.2;
        player.position.z -= 0.1; // Simulate running
        
        // Increase score based on distance traveled
        score += 1;
        scoreElement.innerText = `Score: ${score}`;
    }
    
    updateGround();
    renderer.render(scene, camera);
}
animate();

// Handle window resizing
window.addEventListener('resize', () => {
    renderer.setSize(window.innerWidth, window.innerHeight);
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
});
