<script setup>
import { ref, defineProps, onMounted, watch } from 'vue'
import { loadYmap } from 'vue-yandex-maps'

const settings = {
    apiKey: '20fd5a26-b740-4542-a82a-f78f6d786e00',
    lang: 'ru_RU',
    coordorder: 'latlong', // Порядок задания географических координат
    debug: true, // Режим отладки
    version: '2.1' // Версия Я.Карт
}

const coord = ref([53.304437, 34.303867])
const zoom = ref(16)
const props = defineProps(['start', 'end'])

onMounted(async () => {
    await loadYmap(settings);
    window.ymaps.ready(() => {
        const map = new window.ymaps.Map('map', {
            center: coord.value,
            zoom: zoom.value,
        });
        watch (() => props.start, () => {
            map.geoObjects.removeAll();
            map.setCenter(props.start);
            map.setZoom(18);
            window.ymaps.route([props.start, props.end], {
                routingMode: 'pedestrian',
                multiRoute: true
            }).then(route => {
                map.geoObjects.add(route);
            });
        });
        watch (() => props.end, () => {
            map.geoObjects.removeAll();
            window.ymaps.route([props.start, props.end], {
                routingMode: 'pedestrian',
                multiRoute: true
            }).then(route => {
                map.geoObjects.add(route);
            });
        });
    });
})
</script>

<template>
    <div id="map"></div>
</template>

<style scoped>
    #map {
        width: 96%;
        height: 480px;
    }
</style>