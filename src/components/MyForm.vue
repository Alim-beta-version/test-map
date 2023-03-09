<template>
    <form @submit.prevent>
        <div class="forPhone">
            <input type="text" placeholder="Откуда" list="arrayOfAudiences"
                v-model="from" @keypress="$event => validate($event)" required>
            <span class="right">→</span>
            <span class="down">↓</span>
            <input type="text" placeholder="Куда" list="arrayOfAudiences"
                v-model.trim="to" @keypress="$event => validate($event)" required>
            <datalist id="arrayOfAudiences">
                <optgroup v-for="ms in mas" :key="ms.id">
                    <option v-for="m in ms.audiences" :key="m.name">{{ m.name }}</option>
                </optgroup>
            </datalist>
        </div>
        <button @click="clickBtn">
            Проложить маршрут
        </button>
    </form>
</template>

<script>
import { array } from './Array.js'

export default {
    data: () => ({
        from: '',
        to: '',
        mas: array
    }),
    methods: {
        clickBtn() {
            this.$emit('ok', this.from, this.to)
        },
        validate(e) {
            let char = String.fromCharCode(e.keyCode);
            if (/^[А-Ба-б0-9]+$/.test(char))
                return true;
            else
                e.preventDefault();
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