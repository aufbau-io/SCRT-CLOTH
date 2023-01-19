<script>
import { onMount, onDestroy } from 'svelte';
import * as THREE from 'three';
// import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls.js';
import * as CANNON from 'cannon-es';

let container, pc, id;
	onDestroy(() => cancelAnimationFrame(id));

const renderer = new THREE.WebGLRenderer({antialias: false});
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setClearColor(0xA3A3A3, 0);

const scene = new THREE.Scene();

let sizes = {
  width: window.innerWidth,
  height: window.innerHeight,
};

onMount(() => {
		container.appendChild(renderer.domElement);
	});

const camera = new THREE.PerspectiveCamera(
    30,
    window.innerWidth / window.innerHeight,
    1,
    2000
);

// const orbit = new OrbitControls(camera, renderer.domElement);
// orbit.enableDamping = true;
// orbit.maxPolarAngle = .5;
// orbit.minPolarAngle = Math.PI * 0.5;
// orbit.minDistance = 3.5;
// orbit.maxDistance = 3.5;
// orbit.minAzimuthAngle = -Math.PI;
// orbit.maxAzimuthAngle = Math.PI;
// orbit.enablePan = false;
// orbit.rotateSpeed = .2;
// orbit.zoomSpeed = .2;

// orbit.enabled = false

camera.position.set(0, 0, 2.5);
camera.lookAt(0, 0, 0);



// const ambientLight = new THREE.AmbientLight(0xffffff, 0);
// scene.add(ambientLight)

// fog
{
  const color = 0x232323;
  const density = 0.12;
  scene.fog = new THREE.FogExp2(color, density);
}

const world = new CANNON.World({
    gravity: new CANNON.Vec3(0, -9.8, 0)
});

const Nx = 12;
const Ny = 12;
const mass = 1;
const clothSize = .9;
const dist = clothSize / Nx;

const shape = new CANNON.Particle();

const particles = [];

for(let i = 0; i < Nx + 1; i++) {
    particles.push([]);
    for(let j = 0; j < Ny + 1; j++) {
        const particle = new CANNON.Body({
            mass: j=== Ny ? 0 : mass,
            shape,
            position: new CANNON.Vec3((i - Nx * 0.5) * dist, (j - Ny * 0.5) * dist, 0),
            velocity: new CANNON.Vec3(0, 0, -0.1 * (Ny - j))
        });
        particles[i].push(particle);
        world.addBody(particle);
    }
}

function connect(i1, j1, i2, j2) {
    world.addConstraint(new CANNON.DistanceConstraint(
        particles[i1][j1],
        particles[i2][j2],
        dist
    ));
}

for(let i = 0; i < Nx + 1; i++) {
    for(let j = 0; j < Ny + 1; j++) {
        if(i < Nx)
            connect(i, j, i + 1, j);
        if(j < Ny)
            connect(i, j, i, j + 1);
    }
}

const clothGeometry = new THREE.PlaneGeometry(1, 1, Nx, Ny);

const clothMat = new THREE.MeshBasicMaterial({
  // side: THREE.DoubleSide,
  // wireframe: true,
  map: new THREE.TextureLoader().load('./throw.jpg')
});

const clothMesh = new THREE.Mesh(clothGeometry, clothMat);
clothMesh.position.y += .02;
scene.add(clothMesh);

const clothWireMat = new THREE.MeshBasicMaterial({
  // side: THREE.DoubleSide,
  wireframe: true,
  map: new THREE.TextureLoader().load('./throw.jpg')
});

const clothWireMesh = new THREE.Mesh(clothGeometry, clothWireMat);
clothWireMesh.position.y += .02;
clothWireMesh.position.z += .5;
scene.add(clothWireMesh);

function updateParticules() {
    for(let i = 0; i < Nx + 1; i++) {
        for(let j = 0; j < Ny + 1; j++) {
            const index = j * (Nx + 1) + i;

            const positionAttribute = clothGeometry.attributes.position;

            const position = particles[i][Ny - j].position;

            positionAttribute.setXYZ(index, position.x, position.y, position.z);

            positionAttribute.needsUpdate = true;
        }
    }
}

/**
 * Particles
 */
// Geometry
const cursor = {};
cursor.x = 0;
cursor.y = 0;

const particlesCount = 400;
const positions = new Float32Array(particlesCount * 3);

for (let i = 0; i < particlesCount; i++) {
  positions[i * 3 + 0] = (Math.random() - 0.5) * 4;
  positions[i * 3 + 1] = (Math.random() - 0.5) * 2;
  positions[i * 3 + 2] = (Math.random() - 0.5) * 6;
}

const particlesGeometry = new THREE.BufferGeometry();
particlesGeometry.setAttribute(
  "position",
  new THREE.BufferAttribute(positions, 3)
);

// Material
const particlesMaterial = new THREE.PointsMaterial({
  // sizeAttenuation: textureLoader,
  color: 0xd0d0d0,
  size: 0.02,
});

// $: particlesMaterial.color = $darkMode ? pink : black;

// Points
const stars = new THREE.Points(particlesGeometry, particlesMaterial);
scene.add(stars);


// Sphere

const sphereSize = .3;
const movementRadius = .15;

const sphereGeometry = new THREE.IcosahedronGeometry(sphereSize);
const sphereMat = new THREE.MeshPhysicalMaterial({  
  roughness: 0,  
  transmission: 1,  
  thickness: 0.5, // Add refraction!
});

const sphereMesh = new THREE.Mesh(sphereGeometry, sphereMat);
scene.add(sphereMesh);

sphereMesh.position.z = 1

const sphereShape = new CANNON.Sphere(sphereSize * 1);
let sphereBody = new CANNON.Body({
    shape: sphereShape
});
world.addBody(sphereBody);

const timeStep = 1 / 60;
const clock = new THREE.Clock();

function animate(time) {

  const elapsedTime = clock.getElapsedTime();

  // camera.position.y = (-scrollY / sizes.height) * 1

  // camera.position.x = movementRadius * Math.sin(time / 1500)
  // camera.position.z = Math.cos(time / 1500) + 3

  // camera.position.x += cursor.x / 100
  // camera.position.y += -cursor.y / 100

  camera.position.x += (cursor.x - camera.position.x * 1) * .03;
	camera.position.y += (-cursor.y - camera.position.y * 1) * .04;

		camera.lookAt(scene.position);

    // Animate camera
    // const parallaxX = cursor.x * 0.5;
    // const parallaxY = -cursor.y * 0.5;
    // cameraGroup.position.x +=
    //   (parallaxX - camera.position.x);
    // cameraGroup.position.y +=
    //   (parallaxY - camera.position.y);

    // Update cloth
    updateParticules();
    world.step(timeStep);

    sphereMesh.rotation.x += 0.004;
    sphereMesh.rotation.y += 0.004;
    sphereMesh.rotation.z += 0.004;

    sphereBody.position.set(
        movementRadius * -Math.sin(elapsedTime / 1.5),
        0,
        movementRadius * -Math.cos(elapsedTime / 1.5) + 1
    );
    sphereMesh.position.copy(sphereBody.position);
    renderer.render(scene, camera);

    // orbit.update();
    id = window.requestAnimationFrame(animate);
}

id = window.requestAnimationFrame(animate);

window.addEventListener('resize', function() {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);

    sizes = {
      width: window.innerWidth,
      height: window.innerHeight,
    };
});

window.addEventListener("mousemove", (event) => {
  cursor.x = event.clientX / sizes.width - 0.5;
  cursor.y = event.clientY / sizes.height - 0.5;
});

</script>

<div bind:this={container} class:geometry={true} />

<style>
	.geometry {
		position: fixed;
		top: 0;
		left: 0;
		overflow: hidden;
	}
</style>
