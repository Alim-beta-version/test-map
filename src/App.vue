<template>
    <div class="all">
        <h1>Навигация по БГТУ</h1>
        <MyForm
            @ok="init"
        />
        <MyMap

        />
    </div>
</template>

<script>
import MyForm from '@/components/MyForm.vue'
import MyMap from '@/components/MyMap.vue'

export default {
    components: { MyForm, MyMap },
    data() {
        return {
            from: '',
            to: ''
        }
    },
    methods: {
        init(from, to) {
            this.from = from,
            this.to = to,
            console.log(from, to)
        }
    }
}
</script>

<style scoped>
    .all {
        display: flex;
        flex-direction: column;
        align-items: center;
    }
</style>