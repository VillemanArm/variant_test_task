<script setup lang="ts">
import { reactive, ref, computed, onMounted, onUpdated, watch } from 'vue'
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/Addons.js';

// defineProps<{
// 	msg: string;
// }>();


const cubeCanvas = ref<HTMLInputElement>() // элемент, в котором будет отображаться 3D элемент

onMounted(() => {
	const scene = new THREE.Scene(); // создание сцены
	scene.add(new THREE.GridHelper( 20, 20, 0x707a87, 0xffffff ))

	const axesHelper = new THREE.AxesHelper( 2 ); // создание объекта, показыващего оси. Принимает длину осей
	scene.add( axesHelper)
	
	const renderer = new THREE.WebGLRenderer({ canvas: cubeCanvas.value}); // создание отрисовщика
	renderer.setClearColor('grey') // установка цвета фона
	renderer.setSize( cubeCanvas.value?.offsetWidth || window.innerWidth, cubeCanvas.value?.offsetHeight || window.innerHeight) // установка размера рендерера
	
	// const geometry = new THREE.BoxGeometry( 1, 1, 1 ); // создание объекта
	// const material = new THREE.MeshStandardMaterial( { color: 'green'} ); // на этом материале отображается изменение цвета
	// const cube = new THREE.Mesh( geometry, material );
	// scene.add( cube );
	// scene.add(cube);
	
	const camera = new THREE.PerspectiveCamera( 70, (cubeCanvas.value?.offsetWidth || window.innerWidth) / (cubeCanvas.value?.offsetHeight || window.innerHeight)); // создание камеры. передаем угол обзора в градусах и соотношение сторон
	camera.position.set(3, 3, 3) // установка позиции камеры x, y, z
	camera.lookAt(scene.position); // направление камеры в центр сцены
	scene.add( camera )
	
	const light = new THREE.DirectionalLight(0xffffff, 1); // создание света
	light.position.set(5, 4, 2);
	scene.add(light)
	
	const loader = new GLTFLoader();
	loader.load(
		'/mashes/geometries/cube.glb', 
		(gltf) => {
			// console.log('success');		
			// console.log(gltf.scene.children[0]);
			const cube = gltf.scene.children[0]
			cube.scale.set(2, 2, 2)
			// cube.position.set(0, 2, 0)
			// cube.material =  new THREE.MeshStandardMaterial( { color: 'green'} )
			const textureLoader = new THREE.TextureLoader();
			const texture = textureLoader.load(
				'/mashes/textures/albedo/albedo-wood.png',
				(texture) => {
					cube.material =  new THREE.MeshStandardMaterial( { map: texture} )
					renderer.render(scene, camera)
				},
				undefined,
				(error) => {
					console.log(error);
				}
			
				);

			scene.add(cube)
			renderer.render(scene, camera)
		},
		undefined,
		(error) => {
			console.log(error);
			
		}

	)

	//renderer.render(scene, camera) // запуск отрисовки, делается после создания всех объектов
	
	

	// const animate = (): void => {
	// 	renderer.render(scene, camera)
	// 	requestAnimationFrame(animate)
	// 	cube.rotation.x += 0.01
	// 	cube.rotation.y += 0.01
	// }

	// animate()

})


</script>

<template>

	<canvas ref="cubeCanvas" class="cubeCanvas">
		
	</canvas>



</template>

<style lang="sass">
	@import '@/assets/styles/constants.sass'
	body
		overflow: hidden

	.cubeCanvas
		width: 100vw
		height: 100vh

</style>





