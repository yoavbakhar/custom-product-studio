<template>
    <div class="viewer-container" ref="containerRef">
        <div v-if="loading" class="loading-overlay">Loading 3D Model...</div>
    </div>
</template>

<script setup lang="ts">
import {ref, onMounted, onBeforeUnmount, watch} from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader.js';

const props = defineProps<{
    color: string;
    roughness: number;
    metalness: number;
}>();

const containerRef = ref<HTMLDivElement | null>(null);
const loading = ref<boolean>(true);

let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let renderer: THREE.WebGLRenderer;
let controls: OrbitControls;
let loadedModel: THREE.Group | null = null;
const managedMaterials: THREE.MeshStandardMaterial[] = [];
let animationFrameId: number;

onMounted(() => {
    if(!containerRef.value) return;

    //1. scene
    scene = new THREE.Scene();
    scene.background = new THREE.Color(0x0f172a);

    //2. camera
    const width = containerRef.value.clientWidth;
    const height = containerRef.value.clientHeight;
    camera = new THREE.PerspectiveCamera(45, width / height, 0.1, 1000);
    camera.position.set(2, 1.5, 3);

    //3. renderer
    renderer = new THREE.WebGLRenderer({antialias: true});
    renderer.setSize(width, height);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    containerRef.value.appendChild(renderer.domElement);

    //4. orbit controls (Allows smooth user interaction with the 3D scene)
    controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    controls.dampingFactor = 0.05;
    controls.maxDistance = 8;
    controls.minDistance = 1;

    const pmremGenerator = new THREE.PMREMGenerator(renderer);
    pmremGenerator.compileEquirectangularShader();

    const rgbeLoader = new RGBELoader();
    rgbeLoader.load(
        'https://raw.githubusercontent.com/mrdoob/three.js/master/examples/textures/equirectangular/royal_esplanade_1k.hdr',
        (texture) => {
            const envMap = pmremGenerator.fromEquirectangular(texture).texture;
            scene.environment = envMap;

            renderer.toneMapping = THREE.ACESFilmicToneMapping;
            renderer.toneMappingExposure = 1.0;
            texture.dispose();
            pmremGenerator.dispose();
        }
    )
    //5. lighting
    const ambientLight = new THREE.AmbientLight(0xffffff, 1.0);
    scene.add(ambientLight);

    const keyLight = new THREE.DirectionalLight(0xffffff, 2.5);
    keyLight.position.set(5, 8, 5);
    scene.add(keyLight);

    const fillLight = new THREE.DirectionalLight(0x38bdf8, 1.5);
    fillLight.position.set(-5, -2, -3);
    scene.add(fillLight);

    //6. load GLTF Product Model (Using a public sample for demonstration purposes)
    const loader = new GLTFLoader();
    const modelUrl = 'https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Models/master/2.0/BoomBox/glTF-Binary/BoomBox.glb';

    loader.load(
        modelUrl,
        (gltf) => {
            loadedModel = gltf.scene;
            loadedModel.scale.set(40, 40, 40);
            loadedModel.position.set(0, -0.2, 0);

            //traverse the model to find all Mesh objects and apply a standard material
            loadedModel.traverse((child) => {
                if((child as THREE.Mesh).isMesh) {
                    const mesh = child as THREE.Mesh;
                    if(mesh.material && (mesh.material as THREE.MeshStandardMaterial).isMeshStandardMaterial)
                    {
                        const mat = mesh.material as THREE.MeshStandardMaterial;
                        // Override with initial props
                        mat.color.set(props.color);
                        mat.roughness = props.roughness;
                        mat.metalness = props.metalness;
                        managedMaterials.push(mat);
                    }
                }
            });

            scene.add(loadedModel);
            loading.value = false;
        },
        (xhr) => {
            console.log((xhr.loaded / xhr.total * 100) + '% loaded');
        },
        (error) => {
            console.error('An error happened while loading the model', error);
            loading.value = false;
        }
    )

    //7. animation loop
    const animate = () => {
        animationFrameId = requestAnimationFrame(animate);
        controls.update();
        renderer.render(scene, camera);
    }
    animate();

    //8. responsive window resize
    window.addEventListener('resize', onWindowResize);
});

//watch state props and update Three.js material properties accordingly
watch(
    () => props.color,
    (newColor) => {
        managedMaterials.forEach((material) => {
            material.color.set(newColor);
        });
    }
);

watch(
    () => props.roughness,
    (val) => {
        managedMaterials.forEach((material) => {
            material.roughness = val;
        });
    }
);

watch(
    () => props.metalness,
    (val) => {
        managedMaterials.forEach((material) => {
            material.metalness = val;
        });
    }
);

const onWindowResize = () => {
    if(!containerRef.value||!renderer||!camera) return;
    const width = containerRef.value.clientWidth;
    const height = containerRef.value.clientHeight;
    camera.aspect = width / height;
    camera.updateProjectionMatrix();
    renderer.setSize(width, height);
};

onBeforeUnmount(() => {
    window.removeEventListener('resize', onWindowResize);
    cancelAnimationFrame(animationFrameId);
    if(controls) controls.dispose();
    if(renderer && renderer.domElement) {
        renderer.dispose();
        containerRef.value?.removeChild(renderer.domElement);
    }
});
</script>

<style scoped lang="scss">
.viewer-container {
    width: 100%;
    height: 100%;
    overflow: hidden;
    position: relative;
    cursor: grab;

    &:active {
        cursor: grabbing;
    }

    .loading-overlay {
        position : absolute;
        inset: 0;
        display: flex;
        align-items: center;
        justify-content: center;
        background-color: rgba(15, 23, 42, 0.85);
        color: #38bdf8;
        font-size: 1.1rem;
        font-weight: 600;
        z-index: 10;
    }
}
</style>