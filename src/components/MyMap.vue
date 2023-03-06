<template>
    <yandex-map :coordinates="coord" :zoom="zoom">
    </yandex-map>
</template>

<script>
import { YandexMap } from 'vue-yandex-maps'

export default {
    components: { YandexMap },
    data: () => ({
        coord: [53.304437, 34.303867],
        zoom: 16,
    }),
}
</script>

<style scoped>
    .yandex-container {
        width: 96%;
        height: 480px;
    }
</style>