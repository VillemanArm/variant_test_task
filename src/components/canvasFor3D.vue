<script setup lang="ts">
import { reactive, ref, computed, onMounted, onUpdated, watch } from 'vue'
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/Addons.js';

// defineProps<{
// 	msg: string;
// }>();


const canvas = ref<HTMLInputElement>() // элемент, в котором будет отображаться 3D элемент
const currentMesh = ref<THREE.Mesh>()

const scene = new THREE.Scene();

const gridHelper = new THREE.GridHelper( 20, 20, 0x707a87, 0xffffff )
scene.add(gridHelper)

const axesHelper = new THREE.AxesHelper( 2 ); // создание объекта, показыващего оси. Принимает длину осей
scene.add( axesHelper)

const camera = new THREE.PerspectiveCamera( 70, (canvas.value?.offsetWidth || window.innerWidth) / (canvas.value?.offsetHeight || window.innerHeight)); // создание камеры. передаем угол обзора в градусах и соотношение сторон
camera.position.set(3, 3, 3) // установка позиции камеры x, y, z
camera.lookAt(scene.position); // направление камеры в центр сцены
scene.add( camera )

const light = new THREE.DirectionalLight(0xffffff, 1); // создание света
light.position.set(5, 4, 2);
scene.add(light)

let renderer: THREE.WebGLRenderer

const createRenderer = () => {
	renderer = new THREE.WebGLRenderer({ canvas: canvas.value}); // создание отрисовщика
	renderer.setClearColor('grey') // установка цвета фона
	renderer.setSize( canvas.value?.offsetWidth || window.innerWidth, canvas.value?.offsetHeight || window.innerHeight) // установка размера рендерера
}

// const renderScene = (): void => {
// 	renderer.render(scene, camera)
// 	requestAnimationFrame(renderScene)
	// cube.rotation.x += 0.01
	// cube.rotation.y += 0.01
// }

// renderScene()

const meshLoader = new GLTFLoader();

const addMesh = (meshPath: string) => { //'/meshes/geometries/cube.glb'
	meshLoader.load(
		meshPath, 
		(gltf) => {
			const mesh: THREE.Object3D = gltf.scene.children[0]
			scene.add(mesh)
			renderer.render(scene, camera)
			currentMesh.value = mesh as THREE.Mesh
		},
		undefined,
		(error) => {
			console.log(error);			
		}
	)
}

const textureLoader = new THREE.TextureLoader();

const addTexture = (texturePath: string) => { //'/meshes/textures/albedo/albedo-wood.png'
	textureLoader.load(
	texturePath,
	(texture) => {
		if (currentMesh.value) {
			currentMesh.value.material =  new THREE.MeshStandardMaterial( { map: texture} )
			renderer.render(scene, camera)
		}
	},
	undefined,
	(error) => {
		console.log(error);
	}
	)
}

// watch(meshes, () => {
// 	console.log(meshes);
// 	console.log('meshes changed');
	
// 	// scene.add(meshes.value[meshes.value.length - 1])
// 	// 	const geometry = new THREE.BoxGeometry( 0.5, 0.5, 0.5 ); // создание объекта
// 	// const material = new THREE.MeshStandardMaterial( { color: 'green'} ); // на этом материале отображается изменение цвета
// 	// const cube = new THREE.Mesh( geometry, material );
// 	// scene.add( cube );
// 	// renderer.render(scene, camera)
	
// }, {deep: true})

onMounted(() => {
	createRenderer()
	renderer.render(scene, camera)

	addMesh('/meshes/geometries/cube.glb')

})

</script>

<template>
	<canvas ref="canvas" class="canvas"></canvas>
</template>

<style lang="sass">
	@import '@/assets/styles/constants.sass'
	body
		overflow: hidden

	.canvas
		width: 100vw
		height: 100vh

</style>





