<script setup>
import { ref } from 'vue'
import MyForm from '@/components/MyForm.vue'
import MyMap from '@/components/MyMap.vue'

const from = ref('')
const to = ref('')

const propsItems = (f, t) => {
    from.value = f
    to.value = t
}
</script>

<template>
    <div class="all">
        <h1>Навигация по БГТУ</h1>
        <MyForm @propsEvent="propsItems" />
        <MyMap :start="from" :end="to"/>
    </div>
</template>

<style scoped>
    .all {
        display: flex;
        flex-direction: column;
        align-items: center;
    }
</style>