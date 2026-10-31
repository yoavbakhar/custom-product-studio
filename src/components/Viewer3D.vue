<template>
    <div class="viewer-container" ref="containerRef"></div>
</template>

<script setup lang="ts">
import {ref, onMounted, onBeforeUnmount, watch} from 'vue';
import * as THREE from 'three';

const props = defineProps<{
    color: string;
    roughness: number;
    metalness: number;
}>();

const containerRef = ref<HTMLDivElement | null>(null);
let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let renderer: THREE.WebGLRenderer;
let cube: THREE.Mesh;
let material: THREE.MeshStandardMaterial;
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
    camera.position.set(3, 2, 4);
    camera.lookAt(0, 0, 0);

    //3. renderer
    renderer = new THREE.WebGLRenderer({antialias: true});
    renderer.setSize(width, height);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    containerRef.value.appendChild(renderer.domElement);

    //4. lighting
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.8);
    scene.add(ambientLight);

    const keyLight = new THREE.DirectionalLight(0xffffff, 2.0);
    keyLight.position.set(5, 5, 5);
    scene.add(keyLight);

    const fillLight = new THREE.DirectionalLight(0x38bdf8, 1.0);
    fillLight.position.set(-5, -2, -2);
    scene.add(fillLight);

    //5. temporary placeholder mesh (product)
    const geometry = new THREE.BoxGeometry(1.5, 1.5, 1.5);
    material = new THREE.MeshStandardMaterial({
        color: new THREE.Color(props.color),
        roughness: props.roughness,
        metalness: props.metalness
    });
    cube = new THREE.Mesh(geometry, material);
    scene.add(cube);

    //6. animation loop
    const animate = () => {
        animationFrameId = requestAnimationFrame(animate);
        cube.rotation.y += 0.005;
        renderer.render(scene, camera);
    }
    animate();

    //7. responsive window resize
    window.addEventListener('resize', onWindowResize);
});

//watch state props and update Three.js material properties accordingly
watch(
    () => props.color,
    (newColor) => {
        if(material) material.color.set(newColor);
    }
);

watch(
    () => props.roughness,
    (val) => {
        if(material) material.roughness = val;
    }
);

watch(
    () => props.metalness,
    (val) => {
        if(material) material.metalness = val;
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
}
</style>