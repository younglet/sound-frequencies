<template>
    <h1 class="text-center font-bold text-2xl p-10 text-blue-500 select-none">
        <span>
           波形发生器
        </span>
    </h1>
    <div class="h-2/3 flex  flex-col items-center justify-center">
        <canvas class="m-12 w-2/3 rounded-xl" ref="canvas" @mousedown="startSound" @mouseup="stopSound" @mousemove="adjustFrequency"/>
        <div class="flex justify-center gap-4">
            <button class="flex items-center justify-center gap-2 border rounded-xl p-2 w-24" :class="{ 'bg-blue-500 text-white': waveType === 'sine' }" @click="setWaveType('sine')">
                <Icon name="ph:wave-sine" />
                正弦波
            </button>
            <button class="flex items-center justify-center gap-2 border rounded-xl p-2 w-24" :class="{ 'bg-blue-500 text-white': waveType === 'square' }" @click="setWaveType('square')">
                <Icon name="ph:wave-square-light"/>
                方波
            </button>
            <button class="flex items-center justify-center gap-2 border rounded-xl p-2 w-24" :class="{ 'bg-blue-500 text-white': waveType === 'sawtooth' }" @click="setWaveType('sawtooth')">
                <Icon name="ph:wave-sawtooth"/>
                锯齿波
            </button>
            <button class="flex items-center justify-center gap-2 border rounded-xl p-2 w-24" :class="{ 'bg-blue-500 text-white': waveType === 'triangle' }" @click="setWaveType('triangle')">
                <Icon name="ph:wave-triangle"/>
                三角波
            </button>
        </div>
    </div>
</template>

<script setup>

const canvas = ref(null);
let audioContext = null;
let oscillator = null;

onMounted(() => {
    audioContext = new AudioContext();
});

let waveType = ref('sine');
let isPlaying = ref(false);
let currentFrequency = ref(0);
let offset = 0;

const startSound = (event) => {
    if (!isPlaying.value) {
        oscillator = audioContext.createOscillator();
        oscillator.type = waveType.value;
        currentFrequency.value = getFrequency(event);
        oscillator.frequency.setValueAtTime(currentFrequency.value, audioContext.currentTime);
        oscillator.connect(audioContext.destination);
        oscillator.start();
        isPlaying.value = true;
        drawWaveform();
    }
};

const stopSound = () => {
    if (isPlaying.value) {
        oscillator.stop();
        oscillator.disconnect();
        oscillator = null;
        isPlaying.value = false;
        currentFrequency.value = 0; // Reset frequency to zero
        clearCanvas();
    }
};

const adjustFrequency = (event) => {
    if (isPlaying.value) {
        currentFrequency.value = getFrequency(event);
        oscillator.frequency.setValueAtTime(currentFrequency.value, audioContext.currentTime);
        drawWaveform();
    }
};

const setWaveType = (type) => {
    waveType.value = type;
    if (isPlaying.value) {
        oscillator.type = type;
        drawWaveform();
    }
};

const getFrequency = (event) => {
    const rect = canvas.value.getBoundingClientRect();
    const y = event.clientY - rect.top;
    const height = rect.height;
    const minFreq = 1;
    const maxFreq = 1500;
    const frequency = minFreq + (maxFreq - minFreq) * Math.pow((1 - y / height), 2);
    return frequency;
};

const drawWaveform = () => {
    const ctx = canvas.value.getContext('2d');
    const width = canvas.value.width;
    const height = canvas.value.height;
    ctx.clearRect(0, 0, width, height);
    ctx.fillStyle = 'black';
    ctx.fillRect(0, 0, width, height);
    ctx.beginPath();
    ctx.moveTo(0, height / 2);
    ctx.lineWidth = 2;
    ctx.strokeStyle = 'lime';
    for (let x = 0; x < width; x++) {
        let y;
        switch (waveType.value) {
            case 'sine':
                y = height / 2 + Math.sin((x + offset) * 2 * Math.PI * oscillator.frequency.value / 100 / width) * height / 4;
                break;
            case 'square':
                y = height / 2 + (Math.sin((x + offset) * 2 * Math.PI * oscillator.frequency.value / 100 / width) > 0 ? 1 : -1) * height / 4;
                break;
            case 'sawtooth':
                y = height / 2 + (1 - 2 * (((x + offset) * oscillator.frequency.value / 100 / width) % 1)) * height / 4; // Changed direction
                break;
            case 'triangle':
                y = height / 2 + (2 * Math.abs(2 * (((x + offset) * oscillator.frequency.value / 100 / width) % 1) - 1) - 1) * height / 4;
                break;
        }
        ctx.lineTo(x, y);
    }
    ctx.stroke();
    ctx.fillStyle = 'lime';
    ctx.font = '18px Arial';
    ctx.textAlign = 'right'; // 设置文字向右对齐
    ctx.fillText(`${currentFrequency.value.toFixed(2)} Hz`, width - 10, height - 10); // 固定x坐标
    offset += 1;
    requestAnimationFrame(drawWaveform);
};

const drawOverlay = () => {
    const ctx = canvas.value.getContext('2d');
    const width = canvas.value.width;
    const height = canvas.value.height;
    ctx.clearRect(0, 0, width, height);
    ctx.fillStyle = 'black';
    ctx.fillRect(0, 0, width, height);
    ctx.beginPath();
    ctx.moveTo(0, height / 2);
    ctx.lineTo(width, height / 2);
    ctx.strokeStyle = 'darkgreen';
    ctx.lineWidth = 2; // Updated line width
    ctx.stroke();
    ctx.fillStyle = 'lime';
    ctx.font = '18px Arial';
    ctx.textAlign = 'right'; // 设置文字向右对齐
    ctx.fillText(`${currentFrequency.value.toFixed(2)} Hz`, width - 10, height - 10); // 固定x坐标
    offset += 1;
    requestAnimationFrame(drawOverlay);
};

const clearCanvas = () => {
    const ctx = canvas.value.getContext('2d');
    ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);
    ctx.fillStyle = 'black';
    ctx.fillRect(0, 0, canvas.value.width, canvas.value.height);
    ctx.beginPath();
    ctx.moveTo(0, canvas.value.height / 2);
    ctx.lineTo(canvas.value.width, canvas.value.height / 2);
    ctx.strokeStyle = 'darkgreen';
    ctx.lineWidth = 2; // Updated line width
    ctx.stroke();
};

onMounted(() => {
    canvas.value.width = window.innerWidth;
    canvas.value.height = window.innerHeight * 0.8;
    clearCanvas();
    drawOverlay();
});
</script>
