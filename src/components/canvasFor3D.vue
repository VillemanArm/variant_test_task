<script setup lang="ts">
import { reactive, ref, computed, onMounted, onUpdated, watch } from 'vue'
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/Addons.js';
import { KTX2Loader } from 'three/examples/jsm/loaders/KTX2Loader.js';
import { REVISION } from "three"
import addMenu from '@/components/addMenu.vue'
import meshMenu from '@/components/meshMenu.vue'



// defineProps<{
// 	msg: string;
// }>();
interface Materials {
	[propName: string]: String
}

interface Textures {
	[propName: string]: Materials
}

export type {Textures}

const THREE_PATH = `https://unpkg.com/three@0.${REVISION}.x`
const geometries: string[] = ['chair', 'cube', 'helmet', 'suzanne']
const textureTypes: string[] = ['albedo', 'metalness', 'normal', 'roughness']
const textureMaterials: string[] = ['leather', 'metal', 'velours', 'wood']
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
	

const canvas = ref<HTMLInputElement>() // элемент, в котором будет отображаться 3D элемент
const selectedMesh = ref<THREE.Mesh>()



const scene = new THREE.Scene();

const gridHelper = new THREE.GridHelper( 20, 20, 0x707a87, 0xffffff )
scene.add(gridHelper)

const axesHelper = new THREE.AxesHelper( 2 ); // создание объекта, показыващего оси. Принимает длину осей
scene.add( axesHelper)

const camera = new THREE.PerspectiveCamera( 70, (canvas.value?.offsetWidth || window.innerWidth) / (canvas.value?.offsetHeight || window.innerHeight)); // создание камеры. передаем угол обзора в градусах и соотношение сторон
camera.position.set(4, 4, 4) // установка позиции камеры x, y, z
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

const addMesh = (meshName: string) => { //'/meshes/geometries/cube.glb'
	meshLoader.load(
		`/meshes/geometries/${meshName}.glb`, 
		(gltf) => {
			const mesh: THREE.Object3D = gltf.scene.children[0]
			scene.add(mesh)
			selectedMesh.value = mesh as THREE.Mesh //scene.getObjectById(mesh.id)
			renderer.render(scene, camera)
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
			renderer.render(scene, camera)
			selectedMesh.value = undefined
		}

	}
}

const ktx2Loader = new KTX2Loader();
ktx2Loader.setTranscoderPath( `${THREE_PATH}/examples/jsm/libs/basis/` );

const setTexture = (textureMaterialPath: string) => { //'/meshes/textures/albedo/albedo-wood.png'
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
			} );

		}else {
			const textureLoader = new THREE.TextureLoader();
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

function selectMesh(event) {
    const mouse = new THREE.Vector2();
    mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
    mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

    const raycaster = new THREE.Raycaster();
    raycaster.setFromCamera(mouse, camera);

    const intersects = raycaster.intersectObjects(scene.children, true);

    if (intersects.length > 0) {
        const selectedObject = intersects[0].object;
		selectedMesh.value = selectedObject as THREE.Mesh		
    }
}


const setMeshPosition = (event: Event, axis: 'x' | 'y' | 'z') => {
	if (selectedMesh.value && event.target) {
		const target = event.target as HTMLInputElement
		selectedMesh.value.position[axis] = +target.value 
		renderer.render(scene, camera)
	}
}

onMounted(() => {
	createRenderer()
	renderer.render(scene, camera)
})

</script>

<template>
	<div class="canvas__wrap">
		<canvas ref="canvas" class="canvas" @click="selectMesh($event)"></canvas>
		<addMenu :geometries="geometries" :addMesh="addMesh"/>
		<meshMenu 
			v-if="selectedMesh" 
			:mesh="selectedMesh"
			:texturesPaths="texturesPaths"
			:setMeshPosition="setMeshPosition"
			:setTexture="setTexture"
			:delMesh="delMesh"
		/>

	</div>
</template>

<style lang="sass">
	@import '@/assets/styles/constants.sass'
	// body
	// 	overflow: hidden

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





