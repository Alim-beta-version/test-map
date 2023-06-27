<script setup>
import { ref } from 'vue'
import MyForm from '@/components/MyForm.vue'
import MyMap from '@/components/MyMap.vue'

let from = ref('')
let to = ref('')
// Получаем данные из MaForm.vue
const propsItems = (f, t) => {
    // Готовим их для отправки в MyMap.vue,
    // если получили не null
    if (f && t) {
        from.value = f;
        to.value = t;
    }
}
</script>

<template>
    <div class="all">
        <h1>Навигация по БГТУ</h1>
        <!-- Получение объектов  из MaForm.vue по событию propsEvent с дальнейшей подготовкой отправки -->
        <MyForm @propsEvent="propsItems" />
        <!-- Отправка данных в MyMap.vue с помощью props -->
        <MyMap :start="from" :end="to" />
    </div>
</template>

<style scoped>
    .all {
        display: flex;
        justify-content: center;
        flex-direction: column;
        align-items: center;
    }
    h1 {
        white-space: nowrap;
    }
</style>