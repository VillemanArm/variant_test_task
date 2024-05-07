<script setup lang='ts'>
import { reactive, ref, computed, onMounted, onUpdated, watch } from 'vue'

defineProps<{
    geometries: string[],
    addMesh: Function,
}>();

const geometrySelect = ref<HTMLInputElement>() 

</script>

<template>

    <div class='add-menu'>
        <select ref="geometrySelect">
            <option 
                v-for="(geometry, index) in geometries" 
                :value="geometry" 
                :key="`${geometry} + ${index}`"
            >
               {{ geometry }}
            </option>
        </select>
        <button 
            @click="addMesh(geometrySelect?.value)"
        >
            Add mesh
        </button>
    </div>

</template>

<style scoped lang='sass'>
    @import '@/assets/styles/constants.sass'

    .add-menu
        width: 300px
        padding: 10px
        position: absolute
        display: flex
        justify-content: space-between
        align-items: center

        font-size: 16px

        background-color: $block-background-color

        &>select
            width: 180px
            height: 25px

            cursor: pointer

        &>button
            padding: 2px 6px

</style>
