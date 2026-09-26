<script setup lang="ts">
import { ref, onMounted, onUnmounted } from "vue";
import XpWindow from "../components/XpWindow.vue";

const sectorColors = [
  "bg-red-500",
  "bg-yellow-500",
  "bg-green-500",
  "bg-blue-500",
  "bg-purple-500",
  "bg-pink-500",
];

const emit = defineEmits(["close"]);

type Question = {
  type: "note" | "interval" | "chord" | "solfeggio";
  question: string;
  options: string[];
  correct: string;
  audio?: string;
};

const questions: Question[] = [
  {
    type: "solfeggio",
    question: "Сколько свойств имеет музыкальный звук?",
    options: ["2", "3", "4", "5", "6", "сам думай"],
    correct: "4",
  },
  {
    type: "solfeggio",
    question: "Какое свойство отличает музыкальный звук от шумового звука?",
    options: [
      "высота",
      "тембр",
      "громкость",
      "Прокопчик",
      "все свойства",
      "никакое из свойств",
    ],
    correct: "высота",
  },
  {
    type: "note",
    question: "Буквенное обозначение звука ми бемоль:",
    options: ["a", "h", "e", "is", "es", "dur"],
    correct: "es",
  },
  {
    type: "interval",
    question: "Какой интервал между До и Соль?",
    options: ["Квинта", "Терция", "Секста", "Септима", "Октава", "Прима"],
    correct: "Квинта",
  },
  {
    type: "solfeggio",
    question: "Какой звук получится, если от До вверх взять кварту?",
    options: ["Фа", "Ми", "Соль", "Ля", "Си", "Ре"],
    correct: "Фа",
  },
  {
    type: "note",
    question: "На дне морском лежит морская...",
    options: ["C", "E", "G", "A#", "Bb", "Никита"],
    correct: "G",
  },
  {
    type: "interval",
    question: "Гармонический вид мажора - это лад, в котором:",
    options: [
      "повышается 4 ступень",
      "понижается 6 ступень",
      "повышается 7 ступень",
      "понижается 2 ступень",
      "повышается 3 ступень",
      "понижается 52 ступень",
    ],
    correct: "понижается 6 ступень",
  },
  {
    type: "solfeggio",
    question: "Бекар обозначает?",
    options: [
      "повышение на плутон",
      "понижение на полутон",
      "удвоение ноты",
      "отмену альтерации",
      "удвоение громкости",
      "удвоение высоты",
    ],
    correct: "отмену альтерации",
  },
  {
    type: "chord",
    question: 'Звуки, энгармонически равные "Es": ',
    options: [
      "geses, f",
      "fis, ges",
      "dis, cisis",
      "do, ceses",
      "cs, 1.6",
      "dis, es",
    ],
    correct: "dis, cisis",
  },
  {
    type: "solfeggio",
    question: "Сколько восьмых длительностей содержится в четвертной с точкой?",
    options: ["2", "3", "4", "5", "6", "n-ное количество"],
    correct: "3",
  },
  {
    type: "interval",
    question: "Главные трезвучия лада:",
    options: [
      "S, R, F",
      "S, D, F",
      "T, S, D",
      "S, D, L",
      "1, 3, 5",
      "U, S, D, T",
    ],
    correct: "T, S, D",
  },
];

const current = ref<Question | null>(null);
const answered = ref(false);
const feedback = ref("");
const selected = ref<number | null>(null);

const score = ref(0);
const questionIndex = ref(0);
const maxRounds = questions.length - 1;
const isFinished = ref(false);

function next() {
  if (questionIndex.value >= maxRounds) {
    isFinished.value = true;
    return;
  }

  const index = Math.floor(Math.random() * questions.length);
  current.value = questions[index];
  answered.value = false;
  feedback.value = "";
  selected.value = null;
  questionIndex.value++;

  if (current.value?.audio) {
    const audio = new Audio(current.value.audio);
    audio.play();
  }
}

function resetGame() {
  score.value = 0;
  questionIndex.value = 0;
  isFinished.value = false;
  next();
}

function selectAnswer(option: string, index: number) {
  if (!current.value || answered.value) return;
  selected.value = index;
  answered.value = true;

  if (option === current.value.correct) {
    feedback.value = "Верно!";
    score.value++;
  } else {
    feedback.value = `Неверно.`;
  }
}

const backgroundMusic = new URL(
  "../assets/audio/soft-piano-loop-192098.mp3",
  import.meta.url,
).href;
const audio = new Audio(backgroundMusic);

let isAudioPlaying = ref(true);

function switchAudio() {
  isAudioPlaying.value = !isAudioPlaying.value;

  if (isAudioPlaying.value) {
    audio.loop = true;
    audio.volume = 1;
    audio.play().catch((e) => {
      console.warn(
        "Не удалось воспроизвести фоновую музыку (возможно, блокировка браузера):",
        e,
      );
      isAudioPlaying.value = false;
    });
  } else {
    audio.pause();
    audio.currentTime = 0;
  }
}

onMounted(() => {
  next();

  if (isAudioPlaying.value) {
    audio.loop = true;
    audio.volume = 1;
    audio.play().catch((e) => {
      console.warn(
        "Не удалось воспроизвести фоновую музыку (возможно, блокировка браузера):",
        e,
      );
      isAudioPlaying.value = false;
    });
  } else {
    audio.pause();
    audio.currentTime = 0;
  }
});

onUnmounted(() => {
  audio.pause();
  audio.currentTime = 0;
});
</script>

<template>
  <XpWindow title="Сольфеджиквиз! v1.0" @close="$emit('close')">
    <div
      class="w-full h-full flex items-center justify-center relative bg-gradient-to-br from-blue-100 via-indigo-200 to-purple-300"
    >
      <div class="w-full flex justify-between items-center text-black px-4">
        <button @click="switchAudio" class="absolute cursor-pointer z-1001">
          <img
            class="h-8 w-8"
            v-if="isAudioPlaying"
            src="../assets/icons/medium-volume.png"
          />
          <img
            class="h-8 w-8"
            v-if="!isAudioPlaying"
            src="../assets/icons/mute.png"
          />
        </button>
        <div class="absolute top-3 left-4 text-lg font-bold text-gray-800 z-20">
          Очки: {{ score }}
        </div>
        <div
          class="absolute top-3 right-4 text-lg font-bold text-gray-800 z-20"
        >
          Вопрос: {{ Math.min(questionIndex, maxRounds) }} / {{ maxRounds }}
        </div>
      </div>

      <div
        class="absolute inset-0 flex items-center justify-center text-center px-4 z-10"
      >
        <div v-if="isFinished" class="text-center">
          <h2 class="text-2xl font-bold mb-2">Игра завершена!</h2>
          <div v-if="score >= 10" class="text-lg font-semibold mb-4">
            <p>Поздравляю! Ты крут(крутая), а остальные – пешки!</p>
          </div>
          <p v-else class="text-lg font-semibold mb-4">
            Результат: {{ score }} из {{ maxRounds }}
          </p>
          <button
            @click="resetGame"
            class="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition cursor-pointer"
          >
            Рестарт?!
          </button>
        </div>
        <div v-else>
          <h2 class="text-xl font-bold mb-2">Вопрос:</h2>
          <p class="text-md font-semibold">{{ current?.question }}</p>
          <p v-if="feedback" class="mt-4 font-bold text-lg">{{ feedback }}</p>

          <button
            v-if="answered"
            @click="next"
            class="mt-4 bg-green-600 text-white px-4 py-2 rounded-lg hover:bg-green-700 transition duration-200 cursor-pointer"
          >
            Следующий
          </button>
        </div>
      </div>

      <div
        v-for="(option, index) in current?.options || []"
        :key="index"
        class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-32 h-10 rounded-lg flex items-center justify-center text-center p-5 cursor-pointer transition duration-200 ease-in-out text-sm font-semibold shadow z-1000 text-white"
        :style="{
          transform: `rotate(${index * 60}deg) translate(0, -220px) rotate(-${index * 60}deg)`,
          transformOrigin: 'center',
        }"
        :class="[
          answered && option === current?.correct
            ? 'bg-green-500 text-white'
            : '',
          answered && selected === index && option !== current?.correct
            ? 'bg-red-500 text-white'
            : '',
          !answered
            ? sectorColors[index % sectorColors.length] +
              ' hover:brightness-110'
            : '',
        ]"
        @click="selectAnswer(option, index)"
      >
        {{ option }}
      </div>
    </div>
  </XpWindow>
</template>
