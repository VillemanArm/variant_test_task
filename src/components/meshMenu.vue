<script setup lang='ts'>
import { reactive, ref, computed, onMounted, onUpdated, watch } from 'vue'
import * as THREE from 'three';
import type { Textures } from '@/components/canvasFor3D.vue'

const { mesh, texturesPaths } = defineProps<{ 
	mesh: THREE.Mesh,
    setMeshPosition: Function,
    setMeshScale: Function,
    setMeshAngle: Function,
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
                <input type="number" step="0.1" :value="mesh.position.x" @input="setMeshPosition($event, 'x')">
                <input type="number" step="0.1" :value="mesh.position.y" @input="setMeshPosition($event, 'y')">
                <input type="number" step="0.1" :value="mesh.position.z" @input="setMeshPosition($event, 'z')">
            </div>
        </div>
        <div class="mesh-menu__row">
            <div class="mesh-menu__row-label">Rotation:</div>
            <div class="mesh-menu__position">
                <input type="number" min="-360" step="1" :value="Math.round(mesh.rotation.x * 180 / Math.PI)" @input="setMeshAngle($event, 'x')">
                <input type="number" min="-360" :value="Math.round(mesh.rotation.y * 180 / Math.PI)" @input="setMeshAngle($event, 'y')">
                <input type="number" min="-360" :value="Math.round(mesh.rotation.z  * 180 / Math.PI)" @input="setMeshAngle($event, 'z')">
            </div>
        </div>
        <div class="mesh-menu__row">
            <div class="mesh-menu__row-label">Scale:</div>
            <div class="mesh-menu__position">
                <input type="number" min="0" step="0.1" :value="mesh.scale.x" @input="setMeshScale($event, 'x')">
                <input type="number" min="0" step="0.1" :value="mesh.scale.y" @input="setMeshScale($event, 'y')">
                <input type="number" min="0" step="0.1" :value="mesh.scale.z" @input="setMeshScale($event, 'z')">
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
            <select v-if="typeof selectedTextureType === 'string'" @change="setTexture(texturesPaths[selectedTextureType][($event.target as HTMLInputElement)?.value] )"> 
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
        
        <button class="mesh-menu__button" @click="delMesh()">
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
        align-items: center

        &>select
            width: 146px

            cursor: pointer

    .mesh-menu__position
        width: 146px
        display: flex
        justify-content: space-between

        &>input
            width: 42px

            text-align: right

    .mesh-menu__button
        margin-top: 10px
        height: 25px

</style>
