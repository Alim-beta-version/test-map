<script setup>
import { ref } from 'vue'
import { YandexMap } from 'vue-yandex-maps'

const coord = ref([53.304437, 34.303867])
const zoom = ref(16)
</script>

<template>
    <yandex-map :coordinates="coord" :zoom="zoom">
    </yandex-map>
</template>

<style scoped>
    .yandex-container {
        width: 96%;
        height: 480px;
    }
</style>