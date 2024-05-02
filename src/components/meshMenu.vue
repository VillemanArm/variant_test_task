<script setup lang='ts'>
import { reactive, ref, computed, onMounted, onUpdated, watch } from 'vue'
import * as THREE from 'three';
//import type {Geometry} from './canvasFor3D.vue'


// defineProps<{
//     mesh: THREE.Mesh,
// }>();

const { mesh } = defineProps({ // получение переменной из пропсов
	mesh: {
		type: THREE.Mesh,
		required: true,
	},
    // textureTypes: {
    //     type: Array,
    //     required: true,
    // },
    // textureMaterials: {
    //     type: Array,
    //     required: true,
    // },
    setMeshPosition: {
        type: Function,
        required: true,
    },
    setTexture: {
        type: Function,
        required: true,
    },
    texturesPaths: {
        type: Object,
        required: true,
    },
});

const textureTypeSelect = ref<HTMLInputElement>() 
const textureMaterialSelect = ref<HTMLInputElement>() 

onMounted(() => {
    
})

</script>

<template>

    <div class='mesh-menu'>
        <div class="mesh-menu__header">Selected mesh</div>
        <div class="mesh-menu__row">
            <div class="mesh-menu__row-label">Position:</div>
            <div class="mesh-menu__position">
                <input type="number" :value="mesh.position.x" @input="setMeshPosition($event, 'x')">
                <input type="number" :value="mesh.position.y" @input="setMeshPosition($event, 'y')">
                <input type="number" :value="mesh.position.z" @input="setMeshPosition($event, 'z')">
            </div>
        </div>
        <div class="mesh-menu__row">
            <div class="mesh-menu__row-label">Texture type:</div>
            <select ref="textureTypeSelect" @change="setTexture(texturesPaths[$event.target.value][textureMaterialSelect.value])">
                <option 
                    v-for="(textureType, index) in Object.keys(texturesPaths)" 
                    :value="textureType" 
                    :key="`${textureType}${index}`"
                >
                    {{ textureType }}
                </option>
            </select>
        </div>
        <div class="mesh-menu__row">
            <div class="mesh-menu__row-label">Texture material:</div>
            <select v-if="textureTypeSelect" ref="textureMaterialSelect" @change="setTexture(texturesPaths[textureTypeSelect.value][$event.target.value])">
                <option 
                    v-for="(textureMaterial, index) in Object.keys(texturesPaths[textureTypeSelect.value])" 
                    :value="textureMaterial" 
                    :key="`${textureMaterial} + ${index}`"
                >
                    {{ textureMaterial }}
                </option>
            </select>
        </div>
        
        <button class="mesh-menu__button"
            @click="() => {
                //if(geometrySelect) addMesh(geometrySelect.value)
            }"
        >
            Delete mesh
        </button>

    </div>

</template>

<style scoped lang='sass'>
    @import '@/assets/styles/constants.sass'

    .mesh-menu
        width: 300px
        padding: 10px
        position: absolute
        display: flex
        flex-direction: column

        font-size: 16px

        background-color: $block-background-color

    .mesh-menu__header
        text-align: center

    .mesh-menu__row
        margin-top: 10px
        display: flex
        justify-content: space-between

        &>select
            width: 140px

            cursor: pointer

    .mesh-menu__position
        width: 140px
        display: flex
        justify-content: space-between

        &>input
            width: 40px

            text-align: right

    .mesh-menu__button
        margin-top: 10px
        height: 25px
        

        




</style>
