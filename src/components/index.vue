<template>
    <div class="main_wrapper">
        <div class="item_wrapper">
            <Timer v-for="field in fields" :id="field.id" :key="field.id" @remove='removeTimer(field.id)' ref="fieldRefs" />
            <div class="new_item" @click="addTimer()">
                <div>
                    <p class="txtx">+</p>
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
.item_wrapper {
    display: grid;
    grid-template-rows: 1fr 1fr 1fr 1fr 1fr 1fr 1fr;
    grid-template-columns: 1fr 1fr 1fr;
    grid-gap: 2vw;
    width: 100%;
}

.new_item {
    border: 1px dashed gray;
    height: 100%;
    width: 300px;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: ease-in-out;
}

.new_item:hover {
    border-color: white;
}

.txtx {
    font-size: 56px;
    font-weight: 200;
}

@media screen and (max-width:990px) {
    .item_wrapper {
        grid-template-columns: 1fr 1fr;
        text-align: center;
    }

    .new_item {
        width: 100%;
    }
}

@media screen and (max-width:511px) {
    .item_wrapper {
        grid-template-columns: 1fr;
        text-align: center;
        row-gap: 20px;
    }

}
</style>