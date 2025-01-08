<template>
    <h1 class="text-center font-bold text-2xl p-10 text-blue-500 select-none">
        <span>
            自定义频率音乐键盘
        </span>
    </h1>
    <div class="flex justify-center mb-4 items-center"  @click="toggleMode = !toggleMode" >
        <span class="mr-2">{{ toggleMode ? '锁定模式' : '演奏模式' }}</span>
        <Icon class="text-gray-500" name="tdesign:gesture-press-filled" v-if="!toggleMode"/>
        <Icon class="text-gray-500" name="ic:baseline-lock" v-if="toggleMode"/>
    </div>
    <div class="grid grid-cols-8 gap-4 items-center justify-center p-8 select-none" v-auto-animate ref="parent">
        <div v-for="(button, index) in buttons" :key="button.id"
            @mousedown="toggleMode ? toggleSineWave(button) : startSineWave(button)"
            @mouseup="!toggleMode && stopSineWave(button)" @mouseleave="!toggleMode && stopSineWave(button)"
            :class="{ 'bg-orange-500': button.isPressed, 'bg-blue-500': !button.isPressed, 'hover:bg-blue-400': !button.isPressed }"
            class="relative h-32 rounded-2xl flex flex-col items-center justify-center shadow-sm group">
            <div class="text-white text-center text-2xl font-bold">
                {{ button.frequency }} Hz
            </div>
            <span class="absolute top-2 right-2 opacity-0 group-hover:opacity-100 transition-opacity">
                <Icon name="basil:cancel-solid" class="bg-white" @click.stop="deleteButton(button.id)" />
            </span>
        </div>
        <div class="h-32 border rounded-2xl flex flex-col items-center justify-center shadow-sm">
            <div class='flex items-center justify-center gap-2'>
                <input type="number" class="border p-1 h-8 w-24 text-right" v-model="new_frequency" />
                <span>Hz</span>
                <Icon name="ic:round-add-box" class="bg-blue-500 text-xl" @click="addButtons" />
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue';
import { onMounted } from 'vue';

const buttons = ref([]);
const new_frequency = ref(440);
const toggleMode = ref(false);
const [parent] = useAutoAnimate();

let audioContext = null;
let oscillators = {};

onMounted(() => {
    audioContext = new AudioContext();
});

const startSineWave = (button) => {
    button.isPressed = true;
    const oscillator = audioContext.createOscillator();
    oscillator.type = "sine";
    oscillator.frequency.setValueAtTime(button.frequency, audioContext.currentTime);
    oscillator.connect(audioContext.destination);
    oscillator.start();
    oscillators[button.id] = oscillator;
};

const stopSineWave = (button) => {
    button.isPressed = false;
    if (oscillators[button.id]) {
        oscillators[button.id].stop();
        oscillators[button.id].disconnect();
        delete oscillators[button.id];
    }
};

const toggleSineWave = (button) => {
    if (button.isPressed) {
        stopSineWave(button);
    } else {
        startSineWave(button);
    }
};

const addButtons = () => {
    if (new_frequency.value < 20 || new_frequency.value > 20000) {
        return alert('请输入20-20000之间的数字');
    }
    const id = Date.now() + Math.random();
    buttons.value.push({ id, frequency: new_frequency.value, isPressed: false });
    new_frequency.value = 0;
    buttons.value.sort((a, b) => a.frequency - b.frequency);
};

const deleteButton = (id) => {
    const buttonIndex = buttons.value.findIndex(button => button.id === id);
    if (buttons.value[buttonIndex].isPressed) {
        stopSineWave(buttons.value[buttonIndex]);
    }
    buttons.value.splice(buttonIndex, 1);
};
</script>