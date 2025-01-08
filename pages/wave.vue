<template>
    <h1 class="text-center font-bold text-2xl p-10 text-blue-500 select-none">
        <span>
            声波可视化
        </span>
    </h1>
    <div>
        <canvas ref="canvas" class="w-full h-96"></canvas>
        <div class="piano-container">
            <div v-for="(note, index) in notes" :key="note.frequency" class="key-wrapper">
                <div class="key" :class="{ 'black-key': note.isBlack, 'active-key': activeKeys.includes(note.key) }"
                    @mousedown="handleMouseDown(note)" @mouseup="handleMouseUp(note)" @mouseleave="handleMouseUp(note)">
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>

const notes = ref([
    { key: 'q', frequency: 261.63, isBlack: false }, // C4
    { key: '2', frequency: 277.18, isBlack: true },  // C#4
    { key: 'w', frequency: 293.66, isBlack: false }, // D4
    { key: '3', frequency: 311.13, isBlack: true },  // D#4
    { key: 'e', frequency: 329.63, isBlack: false }, // E4
    { key: 'r', frequency: 349.23, isBlack: false }, // F4
    { key: '5', frequency: 369.99, isBlack: true },  // F#4
    { key: 't', frequency: 392.00, isBlack: false }, // G4
    { key: '6', frequency: 415.30, isBlack: true },  // G#4
    { key: 'y', frequency: 440.00, isBlack: false }, // A4
    { key: '7', frequency: 466.16, isBlack: true },  // A#4
    { key: 'u', frequency: 493.88, isBlack: false }, // B4
    { key: 'z', frequency: 523.25, isBlack: false }, // C5
    { key: 's', frequency: 554.37, isBlack: true },  // C#5
    { key: 'x', frequency: 587.33, isBlack: false }, // D5
    { key: 'd', frequency: 622.25, isBlack: true },  // D#5
    { key: 'c', frequency: 659.25, isBlack: false }, // E5
    { key: 'v', frequency: 698.46, isBlack: false }, // F5
    { key: 'g', frequency: 739.99, isBlack: true },  // F#5
    { key: 'b', frequency: 783.99, isBlack: false }, // G5
    { key: 'h', frequency: 830.61, isBlack: true },  // G#5
    { key: 'n', frequency: 880.00, isBlack: false }, // A5
    { key: 'j', frequency: 932.33, isBlack: true },  // A#5
    { key: 'm', frequency: 987.77, isBlack: false }  // B5
]);

let audioContext = null;
const oscillators = {};
const activeKeys = ref([]);
const canvas = ref(null);
const points = ref([]);
const size = [100, 30]
let canvasContext = null;

const playNote = (frequency, key) => {
    if (!audioContext) {
        audioContext = new AudioContext();
    }
    const oscillator = audioContext.createOscillator();
    oscillator.frequency.setValueAtTime(frequency, audioContext.currentTime);
    oscillator.connect(audioContext.destination);
    oscillator.start();
    oscillators[key] = oscillator;
    activeKeys.value.push(key);
};

const stopNote = (key) => {
    if (oscillators[key]) {
        oscillators[key].stop();
        oscillators[key].disconnect();
        delete oscillators[key];
        activeKeys.value = activeKeys.value.filter(k => k !== key);
    }
};

const handleMouseDown = (note) => {
    playNote(note.frequency, note.key);
};

const handleMouseUp = (note) => {
    stopNote(note.key);
};

const handleKeyDown = (event) => {
    const note = notes.value.find(note => note.key === event.key);
    if (note && !oscillators[event.key]) {
        event.preventDefault();
        playNote(note.frequency, event.key);
    }
};

const handleKeyUp = (event) => {
    const note = notes.value.find(note => note.key === event.key);
    if (note) {
        event.preventDefault();
        stopNote(event.key);
    }
};

const drawGrid = (time) => {
    if (canvasContext) {
        canvasContext.clearRect(0, 0, canvas.value.width, canvas.value.height);
        const [cols, rows] = size;
        let idx, x_, y_;
        let frequencies = Object.values(oscillators).map(oscillator => oscillator.frequency.value);

        for (let y = 0; y < rows; y++) {
            for (let x = 0; x < cols; x++) {
                canvasContext.beginPath();
                idx = y * cols + x;
                x_ = points.value[idx].x;
                if (frequencies.length > 0) {
                    y_ = points.value[idx].y
                    for (let i = 0; i < frequencies.length; i++) {
                        y_ += Math.sin((time + y) * (frequencies[i] / 200)) * 10
                    }
                } else {
                    y_ = points.value[idx].y
                }
                canvasContext.arc(x_, y_, 2, 0, Math.PI * 2);
                canvasContext.fillStyle = 'rgb(0, 150, 200)';
                canvasContext.fill();
            }
        }
    }
}

onMounted(() => {
    const [cols, rows] = size;
    const dpr = window.devicePixelRatio || 1;
    canvas.value.width = canvas.value.clientWidth * dpr;
    canvas.value.height = canvas.value.clientHeight * dpr;
    canvasContext = canvas.value.getContext('2d');
    canvasContext.scale(dpr, dpr);

    for (let y = 0; y < rows; y++) {
        for (let x = 0; x < cols; x++) {
            points.value.push(
                {
                    x: -50 + 2 * x * canvas.value.clientWidth / cols,
                    y: -50 + 2 * y * canvas.value.clientHeight / rows
                });
        }
    }
    let time = 0;
    setInterval(() => { drawGrid(time); time += 0.1 }, 100);
    window.addEventListener('keydown', handleKeyDown);
    window.addEventListener('keyup', handleKeyUp);
});

onBeforeUnmount(() => {
    window.removeEventListener('keydown', handleKeyDown);
    window.removeEventListener('keyup', handleKeyUp);
});
</script>

<style scoped>
.piano-container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 30vh;
    position: relative;
}

.key-wrapper {
    position: relative;
}

.key {
    width: 40px;
    height: 200px;
    background-color: white;
    border: 1px solid black;
    box-sizing: border-box;
    margin: 1px;
    cursor: pointer;
    position: relative;
    z-index: 0;
    transition: background-color 0.2s;
}

.key:hover {
    background-color: #f0f0f0;
}

.key:active,
.active-key {
    background-color: orange;
    border: 0px;
}

.black-key {
    width: 30px;
    height: 120px;
    background-color: black;
    border: 0px solid black;
    position: absolute;
    top: 0;
    z-index: 1;
    transition: background-color 0.2s;
}

.black-key:hover {
    background-color: #333;
}

.black-key:active,
.black-key.active-key {
    background-color: orange;
}

.key-wrapper:nth-child(2) .black-key,
.key-wrapper:nth-child(4) .black-key,
.key-wrapper:nth-child(7) .black-key,
.key-wrapper:nth-child(9) .black-key,
.key-wrapper:nth-child(11) .black-key,
.key-wrapper:nth-child(14) .black-key,
.key-wrapper:nth-child(16) .black-key,
.key-wrapper:nth-child(19) .black-key,
.key-wrapper:nth-child(21) .black-key,
.key-wrapper:nth-child(23) .black-key {
    left: -20px;
    top: -102px;
}
</style>
