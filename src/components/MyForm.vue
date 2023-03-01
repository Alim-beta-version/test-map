<template>
    <form @submit.prevent>
        <div>
            <input type="text" placeholder="Откуда"
                v-model="from">
             → 
            <input type="text" placeholder="Куда"
                v-model="to">
        </div>
        <span v-if="exception">Заполните все поля!</span>
        <button @click="buildRoute">
            Проложить маршрут
        </button>
    </form>
</template>

<script>
export default {
    data() {
        return {
            from: '',
            to: '',
            exception: false
        }
    },
    methods: {
        buildRoute() {
            if (this.from.trim() && this.to.trim()) {
                this.$emit('ok', this.from, this.to)
                this.from = '',
                this.to = '',
                this.exception = false;
            }
            else
                this.exception = true;
        }
    }
}
</script>

<style scoped>
    form {
        display: flex;
        flex-direction: column;
        align-items: center;
    }
    input {
        padding: 10px 15px;
        border: 1px gray solid;
        border-radius: 10px;
    }
    span {
        color: red;
        font-weight: bold;
        font-size: 20px;
        margin-top: 20px;
    }
    button {
        width: max-content;
        margin: 20px 0;
        padding: 10px 20px;
        background: none;
        color: teal;
        border: 2px gray solid;
        border-radius: 10px;
    }
    button:hover {
        background-color: gainsboro;
        color: black;
        border: 2px black solid;
    }
</style>