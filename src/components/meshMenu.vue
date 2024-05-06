<script setup lang='ts'>
import { reactive, ref, computed, onMounted, onUpdated, watch } from 'vue'
import * as THREE from 'three';
import type { Textures } from '@/components/canvasFor3D.vue'

// defineProps<{
//     mesh: THREE.Mesh,
// }>();

const { mesh, texturesPaths } = defineProps<{ 
	mesh: THREE.Mesh,
    setMeshPosition: Function,
    setTexture: Function,
    texturesPaths: Textures,
    delMesh: Function,
}>();

const selectedTextureType = ref<String>()

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
            <select v-model="selectedTextureType" > 
                <option value=""></option>
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
            <select v-if="typeof selectedTextureType === 'string'" @change="setTexture(texturesPaths[selectedTextureType][$event.target?.value])">
                <option value=""></option>
                <option 
                    v-if="selectedTextureType"
                    v-for="(textureMaterial, index) in Object.keys(texturesPaths[selectedTextureType])" 
                    :value="textureMaterial" 
                    :key="`${textureMaterial} + ${index}`"
                >
                    {{ textureMaterial }}
                </option>
            </select>
        </div>
        
        <button class="mesh-menu__button"
            @click="delMesh()"
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
