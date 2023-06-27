<script setup>
import { ref, defineProps, onMounted, watch } from 'vue'
import { loadYmap } from 'vue-yandex-maps'
// По названию переменных и методов всё понятно
const settings = {
    apiKey: 'ca6cff09-194f-4e2b-89cc-3f6ce771a38e',
    lang: 'ru_RU',
    coordorder: 'latlong', // Порядок задания географических координат
    debug: false, // Режим отладки
    version: '2.1' // Версия Яндекс карт
}
const coord = ref([53.304437, 34.303867])
const zoom = ref(16)
// Получаем данные из App.vue
const props = defineProps(['start', 'end'])
let isModalOpen = ref(false)

onMounted(async () => {
    await loadYmap(settings);

    window.ymaps.ready(() => {
        // Создаём карту и привязываем к <div id="map">
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
                routePanelControl.routePanel.state.set({ type: 'pedestrian', });
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
            // Удаляем все прошлые маркеры и выключаем режим пользовательского маршрута
            map.geoObjects.removeAll();
            map.controls.remove(routePanelControl);
            routeButton.deselect();
            isModalOpen.value = false;
            isRoutePanelShow = false;

            const multiRoute = new window.ymaps.multiRouter.MultiRoute({
                referencePoints: [props.start.coords, props.end.coords],
                params: {
                    // Режим: пешеход
                    routingMode: 'pedestrian',
                }
            }, {
                // Скрывание стартовой метки
                wayPointStartVisible: false,
                // Скрывание точки под метками
                pinVisible: false,
                // Автоматически устанавливаем границы карты так, чтобы маршрут был виден целиком
                boundsAutoApply: true,
                // Цвет маршрутной полосы
                routeActiveStrokeColor: "#ff0000",
                // Добавляем надписи на метку
                wayPointFinishIconContentLayout: window.ymaps.templateLayoutFactory.createClass('Войти'),
            });
            // Остановка выполнения всех событий при клике на маркеры +
            // добавление своего функционала: открытие модальное окна с маршрутом на этаже
            multiRoute.getWayPoints().events.add('click', function (e) {
                e.stopImmediatePropagation();
                isModalOpen.value = true;
            }, null, 1);
            // Добавление мультимаршрута на карту
            map.geoObjects.add(multiRoute);
        }
    });
})
// Ниже по названию всё понятно
const closeModal = () => {
    isModalOpen.value = false;
}

let isPanoramaOpen = ref(false)

const openClosePanorama = () => {
    isModalOpen.value = !isModalOpen.value;
    isPanoramaOpen.value = !isPanoramaOpen.value;
}

let forward = ref(true), back = ref(true), left = ref(true), right = ref(true)
let currentIndex = ref(0)

/* const checkImg = () => {
    try {
        require(`@/img/panorama/corpus 2/floor 4/${currentIndex.value + 1}.jpg`);
        forward.value = true;
    } catch (e) {
        forward.value = false;
    }
    try {
        require(`@/img/panorama/corpus 2/floor 4/${currentIndex.value - 1}.jpg`);
        back.value = true;
    } catch (e) {
        back.value = false;
    }
    try {
        require(`@/img/panorama/corpus 2/floor 4/${currentIndex.value + 100}.jpg`);
        left.value = true;
    } catch (e) {
        left.value = false;
    }
    try {
        require(`@/img/panorama/corpus 2/floor 4/${currentIndex.value - 100}.jpg`);
        right.value = true;
    } catch (e) {
        right.value = false;
    }
} */
</script>

<template>
    <!-- Сама карта внутри div -->
    <div id="map"></div>
    <!-- Модальное окно с маршрутом по этажу -->
    <div v-if="isModalOpen" class="modal" @click.self="closeModal">
        <div class="modal-content">
            <span class="panorama" @click="openClosePanorama">Режим панорамы</span>
            <span class="close" @click="closeModal">&times;</span>
            <img :src="require('@/img/floorRoute/corpus ' + props.end.corpusNum + '/floor ' + props.end.floor + '/' +
             props.end.audience + '.svg')" alt="План этажа не найден">
        </div>
    </div>
    <!-- Модальное окно с панорамой -->
    <div v-if="isPanoramaOpen" class="modal" @click.self="openClosePanorama">
        <div class="modal-content">
            <span class="back" @click="openClosePanorama">&larr;</span>
            <img :src="require(`@/img/panorama/corpus ${props.end.corpusNum}/floor ${props.end.floor}/${currentIndex}.jpg`)" alt="План этажа не найден">
            <div class="arrow-block">
                <div @click="forward ? currentIndex++ : currentIndex" class="arrow-forward" :style="{ opacity: forward ? 1 : 0 }"></div>
                <div>
                    <div @click="left ? currentIndex+=100 : currentIndex" class="arrow-left" :style="{ opacity: left ? 1 : 0 }"></div>
                    <div @click="right ? currentIndex-=100 : currentIndex" class="arrow-right" :style="{ opacity: right ? 1 : 0 }"></div>
                </div>
                <div @click="back ? currentIndex-- : currentIndex" class="arrow-backward" :style="{ opacity: back ? 1 : 0 }"></div>
            </div>
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
    margin-bottom: -4px;
    width: 100%;
    border-radius: 30px;
}
.close {
    position: absolute;
    padding: 0 10px;
    right: 10px;
    top: 10px;
    color: red;
    font-size: 35px;
    font-weight: bold;
    border-radius: 50%;
}
.panorama {
    position: absolute;
    padding: 8px 15px;
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
    color: red;
    font-size: 35px;
    font-weight: bold;
    border-radius: 20px;
}
.close:hover, .close:focus, .panorama:hover, .panorama:focus, .back:hover, .back:focus {
    color: black;
    cursor: pointer;
    background-color: gainsboro;
}
.arrow-block {
    position: absolute;
    bottom: 10%;
    left: 0;
    right: 0;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
}
.arrow-forward {
    border-style: solid;
    border-width: 24px;
    border-radius: 50%;
    border-color: transparent transparent #ff0000 transparent;
    transform: rotateX(-55deg);
}
.arrow-backward {
    border-style: solid;
    border-width: 24px;
    border-radius: 50%;
    border-color: #ff0000 transparent transparent transparent;
    transform: rotateX(-45deg);
}
.arrow-left {
    margin-right: 50px;
    display: inline-block;
    border-style: solid;
    border-width: 33px;
    border-radius: 50%;
    border-color: transparent #ff0000 transparent transparent;
    transform: rotateX(-70deg);
}
.arrow-right {
    margin-left: 50px;
    display: inline-block;
    border-style: solid;
    border-width: 33px;
    border-radius: 50%;
    border-color: transparent transparent transparent #ff0000;
    transform: rotateX(-70deg);
}

@media (max-width: 700px) {
    .panorama {
        top: 9px;
        font-size: 16px;
        padding: 7px 14px;
    }
}
@media (max-width: 600px) {
    .close {
        font-size: 28px;
        padding: 0 8px;
    }
    .panorama {
        top: 7px;
        font-size: 10px;
        padding: 5px 12px;
    }
}
@media (max-width: 500px) {
    .close {
        font-size: 20px;
        padding: 0 6px;
    }
    .panorama {
        top: 5px;
        font-size: 8px;
        padding: 3px 10px;
    }
    .back {
        font-size: 28px;
        padding: 0 8px 3px;
    }
    .arrow-forward, .arrow-backward {
        border-width: 15px;
    }
    .arrow-left, .arrow-right {
        border-width: 20px;
    }
}
</style>