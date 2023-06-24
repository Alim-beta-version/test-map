<script setup>
import { ref, defineProps, onMounted, watch } from 'vue'
import { loadYmap } from 'vue-yandex-maps'

const settings = {
    apiKey: 'ca6cff09-194f-4e2b-89cc-3f6ce771a38e',
    lang: 'ru_RU',
    coordorder: 'latlong', // Порядок задания географических координат
    debug: false, // Режим отладки
    version: '2.1' // Версия Я.Карт
}
const coord = ref([53.304437, 34.303867])
const zoom = ref(16)
const props = defineProps(['start', 'end'])
let isModalOpen = ref(false)

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
        // Создаем кнопку с накладыванием слоя панорамы
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
        watch (() => props.start.coords, () => {
            if (props.end.coords === prevEnd)
                buildRoute();
        });
        // Наблюдатель за конечным положением (куда)
        watch (() => props.end.coords, () => {
            prevEnd = props.end.coords;
            buildRoute();
        });
        // Лаконичный метод построения красивого и очень точного маршрута
        const buildRoute = () => {
            map.geoObjects.removeAll();
            map.controls.remove(routePanelControl);
            routeButton.deselect();
            isRoutePanelShow = false;

            const multiRoute = new window.ymaps.multiRouter.MultiRoute({
                referencePoints: [props.start.coords, props.end.coords],
                params: {
                    // Режим: пешеход
                    routingMode: 'pedestrian',
                }
            }, {
                wayPointStartVisible: false,
                pinVisible: false,
                // Автоматически устанавливать границы карты так, чтобы маршрут был виден целиком.
                boundsAutoApply: true,
                // Цвет маршрутной полосы
                routeActiveStrokeColor: "#ff0000",
                wayPointFinishIconContentLayout: window.ymaps.templateLayoutFactory.createClass('Войти'),
            });
            // Остановка выполнения всех событий при клике на маркеры
            multiRoute.getWayPoints().events.add('click', function (e) {
                e.stopImmediatePropagation();
                isModalOpen.value = true;
            }, null, 1);
            map.geoObjects.add(multiRoute);
        }
    });
})

const closeModal = () => {
    isModalOpen.value = false;
}

let isPanoramaOpen = ref(false)

const openClosePanorama = () => {
    isModalOpen.value = !isModalOpen.value;
    isPanoramaOpen.value = !isPanoramaOpen.value;
}
</script>

<template>
    <div id="map"></div>
    <div v-if="isModalOpen" class="modal" @click.self="closeModal">
        <div class="modal-content">
            <span class="panorama" @click="openClosePanorama">Режим панорамы</span>
            <span class="close" @click="closeModal">&times;</span>
            <img :src="require('@/img/' + 'screen' + '.jpg')" alt="План этажа не найден">
        </div>
    </div>
    <div v-if="isPanoramaOpen" class="modal" @click.self="openClosePanorama">
        <div class="modal-content">
            <span class="back" @click="openClosePanorama">&larr;</span>
            <img :src="require('@/img/logo.png')" alt="План этажа не найден">
        </div>
    </div>
</template>

<style scoped>
#map {
    width: 98%;
    height: 500px;
}
.modal {
    display: flex;
    align-items: center;
    justify-content: center;
    position: fixed;
    z-index: 1;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    overflow: auto;
    background-color: rgba(0, 0, 0, 0.5);
}

.modal-content {
    position: relative;
    margin: auto 3%;
    background-color: #fff;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2), 0 16px 20px rgba(0, 0, 0, 0.2);
    border-radius: 30px;
}
img {
    width: 100%;
    border-radius: 30px;
}
.close {
    position: absolute;
    padding: 0 10px;
    right: 10px;
    top: 10px;
    color: gray;
    font-size: 35px;
    font-weight: bold;
    border-radius: 50%;
}
.panorama {
    position: absolute;
    padding: 10px 15px;
    left: 10px;
    top: 10px;
    color: gray;
    font-size: 20px;
    border-radius: 20px;
}
.back {
    position: absolute;
    padding: 0 10px 5px;
    right: 10px;
    top: 10px;
    color: gray;
    font-size: 35px;
    font-weight: bold;
    border-radius: 20px;
}
.close:hover, .close:focus, .panorama:hover, .panorama:focus, .back:hover, .back:focus {
    color: black;
    cursor: pointer;
    background-color: gainsboro;
}
</style>