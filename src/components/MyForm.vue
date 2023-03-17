<script setup>
import { ref } from 'vue'
import { items } from './Array.js'

const from = ref('')
const to = ref('')

const validate = (event) => {
    let char = event.key;
    if (/^[А-Ба-б0-9]+$/.test(char))
        return true;
    else
        event.preventDefault();
}
</script>

<template>
    <form @submit.prevent>
        <div class="forPhone">
            <input type="text" placeholder="Откуда" list="arrayOfAudiences"
                v-model.lazy="from" @keydown="validate" required />
            <span class="right">→</span>
            <span class="down">↓</span>
            <input type="text" placeholder="Куда" list="arrayOfAudiences"
                v-model.lazy="to" @keydown="validate" required />
            <datalist id="arrayOfAudiences">
                <optgroup v-for="ms in items" :key="ms.id">
                    <option v-for="m in ms.audiences" :key="m.name">{{ m.name }}</option>
                </optgroup>
            </datalist>
        </div>
        
        <button @click="$emit('propsEvent', from, to)">Проложить маршрут</button>
    </form>
</template>

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
        font-size: 20px;
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
        box-shadow: 0 0 10px 1px gray;
    }
    .forPhone {
        display: flex;
        flex-direction: row;
        align-items: center;
    }
    .right {
        display: block;
        margin: 0 10px;
    }
    .down {
        display: none;
    }
    @media (max-width: 500px) {
        .forPhone {
            flex-direction: column;
        }
        .right {
            display: none;
        }
        .down {
            display: block;
            margin: 5px 0;
        }
    }
</style>