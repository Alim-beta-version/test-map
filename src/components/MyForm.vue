<script setup>
import { ref } from 'vue'
import { items } from './Array.json'
// По названию переменных и методов всё понятно
let from = ref('')
let to = ref('')
let errorText = ref('')

const swap = () => {
    [from.value, to.value] = [to.value, from.value];
}

const validate = (val) => {
    // Правило валидации
    const regex = /^[а-яА-Я0-9 ,]*$/;
    const isValid = regex.test(val);

    if (!isValid && val)
        errorText.value = "Некорректный ввод. Поля ограничены кириллицей и цифрами (примеры: а228, д, 137, Столовая)";
    else
        errorText.value = '';
}
// Нахождение аудитории по введённым данным в input поле
// и возвращение объекта для отправления в App.vue -> MyMap.vue,
// иначе составляем текст ошибки и возвращаем null
const find = (target) => {
    const fromUper = from.value.toUpperCase();
    const toUpper = to.value.toUpperCase();

    for (let corpus of items)
        for (let element of corpus.floor)
            for (let audit of element.audiences)
                if (audit.name.toUpperCase() === target.toUpperCase() && fromUper !== toUpper)
                    return {
                        corpusNum: corpus.id - 1,
                        coords: corpus.coords,
                        floor: element.num,
                        audience: audit.name,
                    };

    if (fromUper && toUpper) {
        if (fromUper === toUpper)
            errorText.value = "Нет смысла в таком маршруте!";
        else
            errorText.value = "Таких мест не найдено, пожалуйста, проверьте правильность введённых данных.";
    }
    else
        errorText.value = "Заполните все обязательные поля!";

    return null;
}
</script>

<template>
    <form @submit.prevent>
        <div class="forPhone">
            <input type="text" placeholder="Откуда" list="arrayOfAudiences"
                v-model="from" @input="validate(from)" required/>
            <span class="arrow" @click="swap">→</span>
            <input type="text" placeholder="Куда" list="arrayOfAudiences"
                v-model="to" @input="validate(to)" required/>
            <datalist id="arrayOfAudiences">
                <optgroup v-for="ms in items" :key="ms.id">
                    <template v-for="m in ms.floor" :key="m.num">
                        <option v-for="audience in m.audiences" :key="audience.name">
                            {{ audience.name }}
                        </option>
                    </template>
                </optgroup>
            </datalist>
        </div>
        <span v-if="errorText" style="color: red; margin-top: 20px; text-align: center;">{{ errorText }}</span>
        <!-- Отправка объектов при нахождении совпадения в массиве -->
        <button @click="$emit('propsEvent', find(from), find(to))">Проложить маршрут</button>
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
    .arrow:hover {
        cursor: pointer;
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