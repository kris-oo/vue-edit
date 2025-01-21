<template>
    <div class="demo">
        <h1>{{ message }}</h1>
        <button @click="count++">Count: {{ count }}</button>
    </div>
</template>

<script setup>
import { ref } from 'vue'
const message = ref('Hello Vue!')
const count = ref(0)
</script>

<style>
.demo {
    text-align: center;
    padding: 20px;
}

h1 {
    color: #42b883;
    margin-bottom: 20px;
}

button {
    padding: 8px 16px;
    background: #42b883;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

button:hover {
    background: #3aa876;
}
</style>