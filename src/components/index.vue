<template>
    <div class="main_wrapper">
        <div class="item_wrapper">
            <Timer v-for="field in fields" :id="field.id" :key="field.id" @remove='removeTimer(field.id)' ref="fieldRefs" />
            <div @click="addTimer()">
                <div>
                    <p>Add another timer</p>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>

import Timer from './Timer.vue'
import { ref } from 'vue'

let timerCount = 0
const fields = ref([
    { id: 0 }
])
const fieldRefs = ref([])

function addTimer() {
    console.log(1)
    timerCount++
    fields.value.push({
        id: timerCount++
    })
}

function removeTimer(id) {
    console.log(id)
    const index = fields.value.findIndex(f => f.id === id)
    console.log(index)
    fields.value.splice(index, 1)
}

</script>
<style lang="css">
.main_wrapper {
    display: flex;
    flex-wrap: wrap;
}
</style>