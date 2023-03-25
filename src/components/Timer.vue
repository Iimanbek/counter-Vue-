



<!--   Here I render every item of-(stopwatchs) then I send all then ti index.vie   -->
<template>
    <div>
        <div class="item" @click='globalClick'>
            <div>
                <div :style="{ width: percentage * 100 + '%' }" v-if='!countingUp'>
                </div>
            </div>
            <div v-if='displayTitle'>
                <p @click='rename'>
                    {{ displayTitle }}
                    <span>
                        <Icon icon="clarity:pencil-solid" :inline="true" />
                    </span>
                </p>
            </div>
            <div>
                <p :class="{ 'timer-blinking': isBlinking }" @click='retime'>
                    <span class="time">{{ display }}</span><span>.{{ displayMs }}</span>
                    <span>
                        <Icon icon="clarity:pencil-solid" :inline="true" />
                    </span>
                </p>
            </div>
            <div class="controls">
                <button type='button' @click='start' v-if='!isStarted'>
                    <Icon icon="clarity:play-solid" :inline="true" />
                    Start
                </button>
                <button type='button' @click='pause' v-else>
                    <Icon icon="clarity:pause-solid" :inline="true" />
                    Pause
                </button>
                <button type='button' @click='stop' v-if='!isStopped'>
                    <Icon icon="clarity:stop-solid" :inline="true" />
                    Stop
                </button>
                <button @click='countup' v-if='!countingUp'>
                    <Icon icon="clarity:plus-circle-solid" :inline="true" />
                    Count Up
                </button>
                <button @click='countdown' v-else :disabled="isStopped && defaultSeconds === 0">
                    <Icon icon="clarity:minus-circle-solid" :inline="true" />
                    Count Down
                </button>
                <button type='button' @click='remove'>
                    <Icon icon="clarity:trash-solid" :inline="true" />
                    Remove
                </button>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue'
import Swal from 'sweetalert2'


const emit = defineEmits(['remove'])

const timeUnitSeconds = [
    1,
    60,
    60 * 60,
    24 * 60 * 60,
]

const convertTimeFromString = (string = '') => {
    const timeUnits = string.split(':').reverse()
    let seconds = 0
    let isNotNaN = false
    timeUnits.forEach((unit, index) => {
        if (timeUnitSeconds[index] && !isNaN(unit)) {
            seconds += unit * timeUnitSeconds[index]
            isNotNaN = true
        }
    })
    if (isNotNaN) return seconds
    else return NaN
}

const props = defineProps(['id'])

let displayTitle = ref('Timer'),
    display = ref('00:00:00'),
    displayMs = ref('000'),
    percentage = ref(0),
    defaultSeconds = 0,
    isStarted = false,
    isStopped = true,
    countingUp = ref(true),
    targetTimestamp = 0,
    pausedDifference = 0,
    isBlinking = false,
    interval = () => { }

function start() {
    let nowTimestamp = Date.now()
    let targetTime = defaultSeconds
    if (pausedDifference) {
        targetTime = pausedDifference
        pausedDifference = 0
    }
    if (countingUp.value) targetTimestamp = nowTimestamp - targetTime
    else targetTimestamp = nowTimestamp + targetTime
    interval = setInterval(() => {
        if (!countingUp.value && targetTimestamp - Date.now() < 1) {
            stop()
            updateDisplay(0)
            isBlinking = true
            if (defaultSeconds === 0) countingUp.value = true
        } else updateDisplay()
    }, 10)
    isStarted = true
    isStopped = false
    isBlinking = false
}

function pause() {
    let nowTimestamp = Date.now()
    pausedDifference = Math.abs(targetTimestamp - nowTimestamp)
    clearInterval(interval)
    updateDisplay()
    isStarted = false
}

function stop() {
    if (isStarted) pause()
    reset()
    updateDisplay()
    isStopped = true
}

function reset() {
    let nowTimestamp = Date.now()
    if (countingUp.value) targetTimestamp = nowTimestamp - defaultSeconds
    else targetTimestamp = nowTimestamp + defaultSeconds
    pausedDifference = 0
}

/**
 * Remove the timer from parent component
 */
function remove() {
    emit('remove', props.id)
}

/**
 * Update both display (hh:mm:ss and miliseconds)
 * @param {number} targetTime 
 */
function updateDisplay(targetTime) {
    let nowTimestamp = Date.now()
    if (targetTime !== undefined) {
        if (countingUp.value) targetTimestamp = nowTimestamp - targetTime
        else targetTimestamp = nowTimestamp + targetTime
    }
    const time = Math.abs(Date.now() - targetTimestamp) / 1000
    const numMs = Math.floor((time % 1) * 1000)
    const numS = Math.floor(time % 60)
    const numM = Math.floor(time / 60) % 60
    const numH = Math.floor(time / 3600) % 24
    const numD = Math.floor(time / 86400)
    let displayTemp = numS.toString().padStart(2, '0')
    displayTemp = numM.toString().padStart(2, '0') + ':' + displayTemp
    displayTemp = numH.toString().padStart(2, '0') + ':' + displayTemp
    if (numD) displayTemp = numD.toString().padStart(2, '0') + ':' + displayTemp
    display.value = displayTemp
    displayMs.value = ("000" + numMs).slice(-3)
    if (!countingUp.value && defaultSeconds) {
        percentage.value = time * 1000 / defaultSeconds
    }
    // console.log(display)
}

/**
 * Change the title
 */
async function rename() {
    // To change the title
    let { value: title } = await Swal.fire({
        title: 'Enter the new title',
        input: 'text',
        inputValue: displayTitle.value,
        showCancelButton: true
    })
    if (title !== undefined) displayTitle.value = title
}

function countup() {
    // Switch to counting up
    countingUp.value = true
    let nowTimestamp = Date.now()
    let difference = Math.abs(targetTimestamp - nowTimestamp)
    targetTimestamp = nowTimestamp - difference
    // updateDisplay()
}

/**
 * Switch timer to counting down
 */
function countdown() {
    if (!countingUp.value && defaultSeconds) {
        percentage.value = time * 1000 / defaultSeconds
    }
    countingUp.value = false
    let nowTimestamp = Date.now()
    let difference = Math.abs(targetTimestamp - nowTimestamp)
    targetTimestamp = nowTimestamp + difference
    // updateDisplay()
}

/**
 * Change the timer's time
 */
async function retime() {
    let { value: input } = await Swal.fire({
        title: 'Enter the new time',
        input: 'text',
        inputValue: defaultSeconds / 1000,
        showCancelButton: true,
        inputValidator: (input) => {
            let inputSecs = convertTimeFromString(input)
            console.log(inputSecs)
            if (Number.isNaN(Number(inputSecs))) {
                return 'Enter a valid value!'
            } else if (inputSecs < 0) {
                return 'Enter a positive number!'
            }
        }
    })

    if (!input) return

    defaultSeconds = convertTimeFromString(input) * 1000
    updateDisplay(defaultSeconds)
    if (pausedDifference) pausedDifference += defaultSeconds
}

function globalClick() {
    isBlinking = false
}
</script>
<style lang="css">
.timer-blinking {
    animation-duration: 1000ms;
    animation-name: blink;
    animation-iteration-count: infinite;
    animation-direction: normal;
}

@keyframes blink {
    from {
        opacity: 1;
    }

    49% {
        opacity: 1;
    }

    51% {
        opacity: 0.25
    }

    to {
        opacity: 0.25;
    }
}

.item {
    height: 100px;
    width: 100%;
    display: flex;
    justify-content: center;
    flex-direction: column;
    align-items: center;
    background: rgba(105, 105, 105, 1);

}

.time {
    font-size: 24px;
    font-weight: 600;

}
</style>
