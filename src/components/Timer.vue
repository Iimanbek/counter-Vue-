



<!--   Here I render every item of-(stopwatchs) then I send all then ti index.vie   -->
<template>
    <div>
        <div class="item" @click='globalClick'>
            <div>
                <div :style="{ width: percentage * 100 + '%' }" v-if='!countingUp'>
                </div>
            </div>
            <div v-if='displayTitle'>
                <p class="edit_item" @click='rename'>
                    {{ displayTitle }}
                    <span :inline="true">
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"
                            style="fill: white;transform: ;msFilter:;">
                            <path
                                d="m7 17.013 4.413-.015 9.632-9.54c.378-.378.586-.88.586-1.414s-.208-1.036-.586-1.414l-1.586-1.586c-.756-.756-2.075-.752-2.825-.003L7 12.583v4.43zM18.045 4.458l1.589 1.583-1.597 1.582-1.586-1.585 1.594-1.58zM9 13.417l6.03-5.973 1.586 1.586-6.029 5.971L9 15.006v-1.589z">
                            </path>
                            <path
                                d="M5 21h14c1.103 0 2-.897 2-2v-8.668l-2 2V19H8.158c-.026 0-.053.01-.079.01-.033 0-.066-.009-.1-.01H5V5h6.847l2-2H5c-1.103 0-2 .897-2 2v14c0 1.103.897 2 2 2z">
                            </path>
                        </svg>
                    </span>
                </p>
            </div>
            <div>
                <p :class="{ 'timer-blinking': isBlinking }" @click='retime'>
                    <span class="time">{{ display }}</span><span>.{{ displayMs }}</span>
                    <span :inline="true">
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"
                            style="fill: white;transform: ;msFilter:;">
                            <path
                                d="m7 17.013 4.413-.015 9.632-9.54c.378-.378.586-.88.586-1.414s-.208-1.036-.586-1.414l-1.586-1.586c-.756-.756-2.075-.752-2.825-.003L7 12.583v4.43zM18.045 4.458l1.589 1.583-1.597 1.582-1.586-1.585 1.594-1.58zM9 13.417l6.03-5.973 1.586 1.586-6.029 5.971L9 15.006v-1.589z">
                            </path>
                            <path
                                d="M5 21h14c1.103 0 2-.897 2-2v-8.668l-2 2V19H8.158c-.026 0-.053.01-.079.01-.033 0-.066-.009-.1-.01H5V5h6.847l2-2H5c-1.103 0-2 .897-2 2v14c0 1.103.897 2 2 2z">
                            </path>
                        </svg>
                    </span>
                </p>
            </div>
            <div class="controls">
                <button type='button' @click='start' v-if='!isStarted'>
                    <div :inline="true"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"
                            style="fill: white;transform: ;msFilter:;">
                            <path d="M7 6v12l10-6z"></path>
                        </svg></div>

                </button>
                <button type='button' @click='pause' v-else>
                    <div :inline="true">
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"
                            style="fill: white;transform: ;msFilter:;">
                            <path d="M8 7h3v10H8zm5 0h3v10h-3z"></path>
                        </svg>
                    </div>
                </button>
                <button type='button' @click='stop' v-if='!isStopped'>
                    <!-- <Icon icon="clarity:stop-solid" :inline="true" /> -->
                    <div :inline="true">
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"
                            style="fill: white;transform: ;msFilter:;">
                            <path d="M7 7h10v10H7z"></path>
                        </svg>
                    </div>
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
                    <div :inline="true">
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"
                            style="fill: white ;transform: ;msFilter:;">
                            <path
                                d="M6 7H5v13a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7H6zm4 12H8v-9h2v9zm6 0h-2v-9h2v9zm.618-15L15 2H9L7.382 4H3v2h18V4z">
                            </path>
                        </svg>
                    </div>
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


async function rename() {
    let { value: title } = await Swal.fire({
        title: 'Enter the new title',
        input: 'text',
        inputValue: displayTitle.value,
        showCancelButton: true
    })
    if (title !== undefined) displayTitle.value = title
}

function countup() {
    countingUp.value = true
    let nowTimestamp = Date.now()
    let difference = Math.abs(targetTimestamp - nowTimestamp)
    targetTimestamp = nowTimestamp - difference
}

function countdown() {
    if (!countingUp.value && defaultSeconds) {
        percentage.value = time * 1000 / defaultSeconds
    }
    countingUp.value = false
    let nowTimestamp = Date.now()
    let difference = Math.abs(targetTimestamp - nowTimestamp)
    targetTimestamp = nowTimestamp + difference
}

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
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background: rgba(105, 105, 105, 1);
}

.time {
    font-size: 24px;
    font-weight: 600;

}

button {
    background: rgba(0, 0, 0, 0);
    color: white;
    height: 100%;
    border: 1px solid white;
}

.controls,
.edit_item {
    display: flex;
    justify-content: center;
    align-items: center;
}
</style>

