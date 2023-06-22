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
        // Удаляем классический поиск места и разные типы карт
        map.controls.remove('searchControl');
        map.controls.remove('typeSelector');
        // Создаем кнопку для модалки построения маршрута на самой карте
        var routeButton = new window.ymaps.control.Button({
            data: { content: "Построение вашего маршрута", },
            options: {
                maxWidth: 240,
                position: {
                    bottom: 40,
                    left: 8,
                },
            },
        });
        // Переменные для показа/скрытия модалки
        let routePanelControl = null;
        let isRoutePanelShow = false;
        // Обработчик нажатия
        routeButton.events.add('click', () => {
            map.geoObjects.removeAll();

            if (isRoutePanelShow) {
                // Удаляем модалку
                map.controls.remove(routePanelControl);
                isRoutePanelShow = false;
            } else {
                // Добавляем модалку
                routePanelControl = new window.ymaps.control.RoutePanel();
                routePanelControl.routePanel.state.set({
                    type: 'pedestrian',
                });
                map.controls.add(routePanelControl);
                isRoutePanelShow = true;
            }
        });
        var panoramaButton = new window.ymaps.control.Button({
            data: { content: "Панорама", },
            options: {
                maxWidth: 100,
                position: {
                    bottom: 40,
                    left: 250,
                },
            },
        });
        // Обработчик нажатия
        panoramaButton.events.add('click', () => {
            map.getPanoramaManager().then(manager => {
                if (manager.isLookupEnabled())
                    // Выключаем режим просмотра панорамы
                    manager.disableLookup();
                else
                    // Включаем режим просмотра панорамы
                    manager.enableLookup();
            });
        });
        // Добавляем кнопки
        map.controls.add(routeButton);
        map.controls.add(panoramaButton);
        // Переменная используется ниже, в наблюдателе начала, чтобы маршрут
        // не строился дважды при изменении каждого поля ("откуда" и "куда")
        let prevEnd = null;
        // Наблюдатель за начальным положением (откуда)
        watch (() => props.start, () => {
            map.setCenter(props.start);
            map.setZoom(18);

            if (props.end === prevEnd)
                buildRoute();
        });
        // Наблюдатель за конечным положением (куда)
        watch (() => props.end, () => {
            prevEnd = props.end;
            buildRoute();
        });
        // Лаконичный метод построения красивого и очень точного маршрута
        const buildRoute = () => {
            map.geoObjects.removeAll();
            map.controls.remove(routePanelControl);
            routeButton.deselect();
            isRoutePanelShow = false;

            window.ymaps.route([props.start, props.end], {
                routingMode: 'pedestrian',
                multiRoute: true,
            }).then(route => {
                map.geoObjects.add(route);
                var points = route.getWayPoints();
                points.options.set('draggable', false);
            });
        }
    });
})
</script>

<template>
    <div id="map"></div>
</template>

<style scoped>
    #map {
        width: 98%;
        height: 500px;
    }
</style>