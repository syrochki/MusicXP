<script setup lang="ts">
defineProps<{
  title: string;
}>();

const emit = defineEmits(["close"]);
</script>

<template>
  <div
    class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-[800px] h-[800px] bg-white border border-gray-500 shadow-xl rounded-xl"
  >
    <div
      class="bg-gradient-to-b from-[#1b3fa2] to-[#3583d6] h-8 text-white flex items-center justify-between px-3 font-bold text-sm rounded-t-lg"
    >
      <span>{{ title }}</span>
      <button
        @click="emit('close')"
        class="bg-red-600 hover:bg-red-800 transition w-8 h-6 text-[16px] font-extrabold rounded-sm leading-none cursor-pointer"
      >
        ✕
      </button>
    </div>

    <div class="w-full h-[calc(100%-2rem)] overflow-auto relative">
      <slot />
    </div>
  </div>
</template>
