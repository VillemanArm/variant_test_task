<script setup lang="ts">
import { reactive, ref, computed, onMounted, onUpdated, watch } from 'vue'
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/Addons.js';
import { KTX2Loader } from 'three/examples/jsm/loaders/KTX2Loader.js';
import { REVISION } from "three"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import addMenu from '@/components/addMenu.vue'
import meshMenu from '@/components/meshMenu.vue'

interface Materials {
	[propName: string]: String
}

interface Textures {
	[propName: string]: Materials
}

export type {Textures}

const THREE_PATH = `https://unpkg.com/three@0.${REVISION}.x`

const geometries: string[] = ['chair', 'cube', 'helmet', 'suzanne'] // список подгружаемых мэшей
const texturesPaths: Textures = { // структура для выбора текстур сделана так чтобы оставаться независимой при добавлении новых текстур
	albedo: {
		leather: '/meshes/textures/albedo/albedo-leather.ktx2', 
		metal: '/meshes/textures/albedo/albedo-metal.png', 
		velours: '/meshes/textures/albedo/albedo-velours.png', 
		wood: '/meshes/textures/albedo/albedo-wood.png'
	}, 
	metalness: {
		leather: '/meshes/textures/metalness/metalness-leather.ktx2', 
		metal: '/meshes/textures/metalness/metalness-metal.png', 
		velours: '/meshes/textures/metalness/metalness-velours.png', 
		wood: '/meshes/textures/metalness/metalness-wood.png'
	}, 
	normal: {
		leather: '/meshes/textures/normal/normal-leather.ktx2', 
		metal: '/meshes/textures/normal/normal-metal.png', 
		velours: '/meshes/textures/normal/normal-velours.png', 
		wood: '/meshes/textures/normal/normal-wood.png'
	}, 
	roughness: {
		leather: '/meshes/textures/roughness/roughness-leather.ktx2', 
		metal: '/meshes/textures/roughness/roughness-metal.png', 
		velours: '/meshes/textures/roughness/roughness-velours.png', 
		wood: '/meshes/textures/roughness/roughness-wood.png'
	}
}
const canvas = ref<HTMLElement>() 
const selectedMesh = ref<THREE.Mesh>()

let scene: THREE.Scene

const createScene = () => {
	scene = new THREE.Scene();
	
	const gridHelper = new THREE.GridHelper( 20, 20, 0x707a87, 0xffffff )
	scene.add(gridHelper)
	
	const axesHelper = new THREE.AxesHelper( 2 ); // создание объекта, показыващего оси. Принимает длину осей
	scene.add( axesHelper)
}

createScene()

let camera: THREE.PerspectiveCamera

const createCamera = () => {	camera = new THREE.PerspectiveCamera( 70, (canvas.value?.offsetWidth || window.innerWidth) / (canvas.value?.offsetHeight || window.innerHeight)); // создание камеры. передаем угол обзора в градусах и соотношение сторон
	camera.position.set(4, 4, 4)
	camera.lookAt(scene.position); 

	const controls = new OrbitControls(camera, renderer.domElement)
	
	scene.add( camera )
	controls.update();
}	

let light: THREE.DirectionalLight

const createLight = () => {
	light = new THREE.DirectionalLight(0xffffff, 1); // создание света
	light.position.set(5, 4, 2);
	scene.add(light)
}

createLight()

let renderer: THREE.WebGLRenderer

const createRenderer = () => {
	renderer = new THREE.WebGLRenderer({ canvas: canvas.value}); // создание отрисовщика
	renderer.setClearColor('grey') // установка цвета фона
	renderer.setSize( canvas.value?.offsetWidth || window.innerWidth, canvas.value?.offsetHeight || window.innerHeight) // установка размера рендерера
}

const renderScene = (): void => {
	renderer.render(scene, camera)
	requestAnimationFrame(renderScene)
	// animation example
	// cube.rotation.x += 0.01
	// cube.rotation.y += 0.01
}

const meshLoader = new GLTFLoader();

const addMesh = (meshName: string) => {
	meshLoader.load(
		`/meshes/geometries/${meshName}.glb`, 
		(gltf) => {		
			selectedMesh.value = gltf.scene.children[0]	as THREE.Mesh
			gltf.scene.children.forEach((mesh) => {
					scene.add(mesh);
				}
			);
		},
		undefined,
		(error) => {
			console.log(error);			
		}
	)
}

const delMesh = () => {
	if (selectedMesh.value) {
		const objectToRemove = scene.getObjectByName(selectedMesh.value.name);
		if (objectToRemove) {
			scene.remove(objectToRemove)
			selectedMesh.value = undefined
			renderer.render(scene, camera)
		}
	}
}

const ktx2Loader = new KTX2Loader(); // создан вне функции, так как иначе возникает ошибка из-за создания нескольких ktx загрузчиков
ktx2Loader.setTranscoderPath( `${THREE_PATH}/examples/jsm/libs/basis/` );

const textureLoader = new THREE.TextureLoader();

const setTexture = (textureMaterialPath: string) => {
	if (textureMaterialPath) {
		if (textureMaterialPath.endsWith('.ktx2')) {
			ktx2Loader.detectSupport(renderer);
			ktx2Loader.load( textureMaterialPath, function ( texture ) {			
				if (selectedMesh.value) {
					selectedMesh.value.material = new THREE.MeshStandardMaterial( { map: texture} )
					renderer.render(scene, camera)
				}
			}, 
			undefined,
			function (error) {
				console.error(error);
			} 
		);
		}else {			
			textureLoader.load(
				textureMaterialPath,
				(texture) => {
					if (selectedMesh.value) {
						selectedMesh.value.material = new THREE.MeshStandardMaterial( { map: texture} )
						renderer.render(scene, camera)
					}
				},
				undefined,
				(error) => {
					console.log(error);
				}
			)
		}
	}	
}

function selectMesh(event: MouseEvent) {
    const mouse = new THREE.Vector2();
    mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
    mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

    const raycaster = new THREE.Raycaster();
    raycaster.setFromCamera(mouse, camera);

    const intersects = raycaster.intersectObjects(scene.children, true);

    if (intersects.length > 0) {
		for (let i = 0; i < intersects.length; i++) {
			if (intersects[i].object instanceof THREE.Mesh) {
				selectedMesh.value = intersects[i].object as THREE.Mesh
				break				
			}
		}
    }
}

const setMeshPosition = (event: Event, axis: 'x' | 'y' | 'z') => {
	if (selectedMesh.value && event.target) {
		const target = event.target as HTMLInputElement
		selectedMesh.value.position[axis] = +target.value 
	}
}

const setMeshScale = (event: Event, axis: 'x' | 'y' | 'z') => {
	if (selectedMesh.value && event.target) {
		const target = event.target as HTMLInputElement
		selectedMesh.value.scale[axis] = +target.value 
	}
}

const setMeshAngle = (event: Event, axis: 'x' | 'y' | 'z') => {
	if (selectedMesh.value && event.target) {
		const target = event.target as HTMLInputElement
		selectedMesh.value.rotation[axis] = +target.value * Math.PI * 2 / 360 
	}
}

onMounted(() => {
	// Для возможности вставки нескольких независимых канвасов на страницу пришлось сделать ссылку 
	// на канвас через ref, а ref определяется во время монтажа компонента
	// Поэтому создание компонентов, которые ссылаются на канвас, вынесено в хук onMounted
	createRenderer()
	createCamera()
	renderScene()
})

</script>

<template>
	<div class="canvas__wrap">
		<canvas 
			ref="canvas"
			class="canvas" 
			@click="selectMesh($event)"
		>
		</canvas>
		<addMenu 
			:geometries="geometries" 
			:addMesh="addMesh"
		/>
		<meshMenu 
			v-if="selectedMesh" 
			:mesh="selectedMesh"
			:texturesPaths="texturesPaths"
			:setMeshPosition="setMeshPosition"
			:setMeshScale="setMeshScale"
			:setMeshAngle="setMeshAngle"
			:setTexture="setTexture"
			:delMesh="delMesh"
		/>

	</div>
</template>

<style lang="sass">
	@import '@/assets/styles/constants.sass'

	.canvas__wrap
		position: relative
		overflow: hidden

	.canvas
		width: 100vw
		height: 100vh
		
	.add-menu
		top: 10px
		right: 10px

	.mesh-menu
		top: 65px
		right: 10px

</style>





