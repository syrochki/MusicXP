<script setup lang="ts">
import { ref, computed, onUnmounted } from "vue";
import { SplendidGrandPiano } from "smplr";
import XpWindow from "../components/XpWindow.vue";

const emit = defineEmits(["close"]);

const intervals = [
  // Малые и большие секунды
  { name: "м2 ↑ от H", notes: ["H4", "C5"] },
  { name: "м2 ↓ от E", notes: ["E4", "Es4"] },
  { name: "Б2 ↑ от F", notes: ["F4", "G4"] },
  { name: "Б2 ↓ от D", notes: ["D4", "C4"] },

  // Малые и большие терции
  { name: "м3 ↑ от C", notes: ["C4", "Es4"] },
  { name: "м3 ↓ от H", notes: ["H4", "Gis4"] },
  { name: "Б3 ↑ от C", notes: ["C4", "E4"] },
  { name: "Б3 ↓ от A", notes: ["A4", "F4"] },

  // Чистые кварты и квинты
  { name: "ч4 ↑ от G", notes: ["G4", "C5"] },
  { name: "ч4 ↓ от H", notes: ["H4", "Fis4"] },
  { name: "ч5 ↑ от F", notes: ["F4", "C5"] },
  { name: "ч5 ↓ от C", notes: ["C5", "F4"] },

  // Тритоны
  { name: "ув4 ↑ от C", notes: ["C4", "Fis4"] },
  { name: "ум5 ↓ от G", notes: ["G4", "Cis4"] },

  // Малые и большие сексты
  { name: "м6 ↓ от A", notes: ["A4", "C4"] },
  { name: "Б6 ↓ от C", notes: ["C5", "E4"] },
  { name: "Б6 ↑ от E", notes: ["E4", "C5"] },

  // Малые и большие септимы
  { name: "м7 ↓ от C", notes: ["C5", "D4"] },
  { name: "Б7 ↓ от C", notes: ["C5", "Cis4"] },

  // Чистая октава
  { name: "ч8 ↓ от C", notes: ["C5", "C4"] },
  { name: "ч8 ↑ от С", notes: ["C4", "C5"] },
];

const noteToPitch: Record<string, number> = {
  C: 0, Cis: 1, Des: 1,
  D: 2, Dis: 3, Es: 3,
  E: 4,
  F: 5, Fis: 6, Ges: 6,
  G: 7, Gis: 8, As: 8,
  A: 9, Ais: 10, B: 10,
  H: 11,
};

const toScientific: Record<string, string> = {
  C: "C", Cis: "C#", Des: "Db",
  D: "D", Dis: "D#", Es: "Eb",
  E: "E",
  F: "F", Fis: "F#", Ges: "Gb",
  G: "G", Gis: "G#", As: "Ab",
  A: "A", Ais: "A#", B: "Bb",
  H: "B",
};

const currentInterval = ref(intervals[Math.floor(Math.random() * intervals.length)]);
const selectedNotes = ref<string[]>([]);
const candidateNote = ref<string | null>(null);
const feedback = ref<"correct" | "wrong" | "">("");
const score = ref(0);
const round = ref(1);
const maxRounds = 10;
const isFinished = ref(false);

const whiteKeys = ["C4", "D4", "E4", "F4", "G4", "A4", "H4", "C5"];

const blackKeys = [
  { note: "Cis4", alt: "Des4", after: 0.25 },
  { note: "Dis4", alt: "Es4", after: 1.25 },
  { note: "Fis4", alt: "Ges4", after: 3.25 },
  { note: "Gis4", alt: "As4", after: 4.25 },
  { note: "Ais4", alt: "B4", after: 5.25 },
];

function getBlackKeyStyle(afterIndex: number) {
  const total = whiteKeys.length; // 8
  // 0.70 — сдвиг от начала белой клавиши
  const leftPercent = ((afterIndex + 0.70) / total) * 100;

  return {
    left: `calc(${leftPercent}% - 3.2%)`,
    width: `${(0.65 / total) * 100}%`,
  };
}

function getBaseName(fullNote: string): string {
  return fullNote.replace(/\d/g, "");
}

function isSelected(fullNote: string) {
  return selectedNotes.value.includes(fullNote);
}

function isCandidate(fullNote: string) {
  return candidateNote.value === fullNote;
}

function isCorrectKey(fullNote: string) {
  return currentInterval.value.notes.some(
    (n) => getPitch(n) === getPitch(fullNote)
  );
}

let piano: SplendidGrandPiano | null = null;
let audioReady = false;
let currentStop: (() => void) | null = null;

async function initAudio() {
  if (audioReady) return;
  try {
    const context = new AudioContext();
    piano = new SplendidGrandPiano(context);
    await context.resume();
    audioReady = true;
  } catch (e) {
    console.warn("error", e);
  }
}

function playNote(fullNote: string) {
  if (!piano || !audioReady) return;

  if (currentStop) {
    currentStop();
    currentStop = null;
  }

  const base = getBaseName(fullNote);
  const octave = fullNote.replace(/\D/g, "");
  const scientific = (toScientific[base] || base) + octave;

  const stop = piano.start({ note: scientific, velocity: 85 });

  const timer = setTimeout(() => {
    stop();
    if (currentStop === stop) currentStop = null;
  }, 1000);

  currentStop = () => {
    clearTimeout(timer);
    stop();
  };
}

async function pressNote(fullNote: string) {
  if (feedback.value || selectedNotes.value.length >= 2) return;

  await initAudio();
  playNote(fullNote);
  candidateNote.value = fullNote;
}

function confirmNote() {
  if (!candidateNote.value || feedback.value) return;
  if (selectedNotes.value.length >= 2) return;
  if (selectedNotes.value.includes(candidateNote.value)) return;

  selectedNotes.value.push(candidateNote.value);
  candidateNote.value = null;

  if (selectedNotes.value.length === 2) {
    checkAnswer();
  }
}

function getPitch(fullNote: string): number {
  const base = getBaseName(fullNote);
  const octave = parseInt(fullNote.replace(/\D/g, "")) || 4;
  return octave * 12 + (noteToPitch[base] ?? 0);
}


function checkAnswer() {
  const correctPitches = currentInterval.value.notes
    .map(getPitch)
    .sort((a, b) => a - b);

  const answerPitches = selectedNotes.value
    .map(getPitch)
    .sort((a, b) => a - b);

  const isCorrect =
    correctPitches.length === answerPitches.length &&
    correctPitches.every((p, i) => p === answerPitches[i]);

  feedback.value = isCorrect ? "correct" : "wrong";
  if (isCorrect) score.value++;
}

function nextInterval() {
  if (isFinished.value) return;

  if (round.value >= maxRounds) {
    isFinished.value = true;
    return;
  }

  selectedNotes.value = [];
  candidateNote.value = null;
  feedback.value = "";
  currentInterval.value = intervals[Math.floor(Math.random() * intervals.length)];
  round.value++;
}

function resetGame() {
  selectedNotes.value = [];
  candidateNote.value = null;
  feedback.value = "";
  score.value = 0;
  round.value = 1;
  isFinished.value = false;
  currentInterval.value = intervals[Math.floor(Math.random() * intervals.length)];
}

function keyClass(fullNote: string) {
  if (feedback.value === "correct") {
    return isSelected(fullNote) ? "!bg-green-500 !text-white" : "";
  }
  if (feedback.value === "wrong") {
    if (isSelected(fullNote)) return "!bg-red-500 !text-white";
    if (isCorrectKey(fullNote)) return "!bg-blue-500 !text-white";
    return "";
  }
  if (isSelected(fullNote)) return "!bg-yellow-300 !text-black";
  if (isCandidate(fullNote)) return "!bg-yellow-300 !text-black";
  return "";
}

const confirmButtonText = computed(() => "Выбор");

onUnmounted(() => {
  if (currentStop) currentStop();
});
</script>

<template>
  <XpWindow title="ФоноФанат2000" @close="$emit('close')">
    <div class="w-full h-full bg-violet-800 p-4 flex flex-col items-center justify-between">
      <!-- Верхняя панель -->
      <div class="w-full flex justify-between items-center text-gray-100 px-4">
        <span class="font-bold text-lg">
          Раунд: {{ Math.min(round, maxRounds) }} из {{ maxRounds }}
        </span>
        <span class="font-bold text-lg">Очки: {{ score }}</span>
      </div>

      <div class="w-full flex flex-col items-center justify-center min-h-[120px] mt-2">
        <div v-if="!isFinished" class="text-gray-100 text-2xl font-bold mb-2 text-center">
          Интервал: {{ currentInterval.name }}
        </div>

        <div v-if="!isFinished && !feedback" class="flex flex-col items-center">
          <div class="text-sm mb-2 h-6 flex"></div>
          <button
            @click="confirmNote"
            :disabled="!candidateNote"
            class="px-4 py-1 rounded font-medium transition cursor-pointer w-24"
            :class="candidateNote
              ? 'bg-blue-600 text-white hover:bg-blue-700'
              : 'bg-gray-300 text-gray-500 cursor-not-allowed'"
          >
            {{ confirmButtonText }}
          </button>
        </div>

        <div v-if="!isFinished && feedback" class="flex flex-col items-center">
          <div class="h-8 mb-2"></div>
          <button
            @click="nextInterval"
            class="px-4 py-1 rounded font-medium transition cursor-pointer w-24 bg-blue-600 text-white hover:bg-blue-700"
          >
            Далее
          </button>
        </div>

        <div v-if="isFinished" class="flex flex-col items-center">
          <h2 class="text-2xl font-bold mb-2 text-gray-100">Игра завершена!</h2>
          <p class="text-lg mb-3 text-gray-100">Ты набрал(а): {{ score }} из {{ maxRounds }}</p>
          <button
            @click="resetGame"
            class="px-4 py-1 rounded font-medium transition cursor-pointer w-24 bg-blue-600 text-white hover:bg-blue-700"
          >
            Рестарт?!
          </button>
        </div>
      </div>

      <div class="relative w-full max-w-3xl h-52 select-none rounded-xl shadow-xl overflow-hidden bg-gray-200">
        <div class="flex h-full">
          <div
            v-for="note in whiteKeys"
            :key="note"
            @click="pressNote(note)"
            class="flex-1 h-full border-r border-gray-400 bg-white cursor-pointer hover:bg-gray-50 active:bg-gray-100 flex items-end justify-center relative z-10 transition-colors duration-150"
            :class="keyClass(note)"
          >
            <span class="mb-3 text-xs font-medium">{{ getBaseName(note) }}</span>
          </div>
        </div>

        <div
          v-for="key in blackKeys"
          :key="key.note"
          :style="getBlackKeyStyle(key.after)"
          @click="pressNote(key.note)"
          class="absolute top-0 h-[58%] z-20 bg-gray-900 rounded-b-md cursor-pointer hover:bg-gray-800 active:bg-gray-700 flex items-end justify-center transition-colors duration-150 shadow-md"
          :class="keyClass(key.note)"
        >
        </div>
      </div>

      <div class="flex text-xs text-white mt-4 flex-wrap justify-center">
      </div>
    </div>
  </XpWindow>
</template>