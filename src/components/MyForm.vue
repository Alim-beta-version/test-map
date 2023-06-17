<script setup>
import { ref, computed } from 'vue'
import { items } from './Array.js'

const from = ref('')
const to = ref('')
const error = ref(false)
const isEmpty = computed(() => {
    return from.value === '' || to.value === '';
});

const validate = (val) => {
    const regex = /^[а-яА-Я1-9][\d]*$/;
    const isValid = regex.test(val);

    if (!isValid && from.value != '' && to.value != '')
        error.value = true;
    else
        error.value = false;        
}

const find = (target) => {
    for (const corpus of items)
        for (const element of corpus.audiences)
            if (element.name === target)
                return corpus.coords;
    return null;
}
</script>

<template>
    <form @submit.prevent>
        <div class="forPhone">
            <input type="text" placeholder="Откуда" list="arrayOfAudiences"
                v-model="from" @input="validate(from)" required />
            <span class="arrow">→</span>
            <input type="text" placeholder="Куда" list="arrayOfAudiences"
                v-model="to" @input="validate(to)" required />
            <datalist id="arrayOfAudiences">
                <optgroup v-for="ms in items" :key="ms.id">
                    <option v-for="m in ms.audiences" :key="m.name">{{ m.name }}</option>
                </optgroup>
            </datalist>
        </div>
        <span v-if="error" style="color: red; margin-top: 20px;">Некорректный ввод. Оба поля обязательны и ограничены символами и их порядком (примеры: а228, Д, 137)</span>
        <button :disabled="isEmpty" @click="$emit('propsEvent', find(from.toUpperCase()), find(to.toUpperCase()))">Проложить маршрут</button>
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
    button {
        width: max-content;
        margin: 20px 0;
        padding: 10px 20px;
        background: none;
        color: black;
        border: 2px gray solid;
        border-radius: 10px;
    }
    button:hover {
        background-color: aqua;
        color: black;
        border-color: aqua;
    }
    .forPhone {
        display: flex;
        flex-direction: row;
        align-items: center;
    }
    .arrow {
        margin: 0 10px;
        font-size: 20px;
    }
    @media (max-width: 500px) {
        .forPhone {
            flex-direction: column;
        }
        .arrow {
            transform: rotate(90deg);
            margin: 5px 0;
        }
    }
</style>