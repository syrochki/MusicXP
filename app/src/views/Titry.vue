<script setup lang="ts">
import XpWindow from "../components/XpWindow.vue";
</script>

<template>
  <XpWindow title="github.com/syrochki" @close="$emit('close')">
    <div class="pt-10">
      <h2 class="text-xl font-bold mb-2 text-center">Титры:</h2>
      <p class="text-gray-700 mb-4 text-center">
        Программист, компьютерных дел мастер: Зыков Тимофей<br />
        Гейм-дизайнер: Шешин Никита<br />
        Дизайнер: Прокопчик Тимофей<br />

        <br>код проекта: <a src="https://github.com/syrochki/MusicXP">https://github.com/syrochki/MusicXP<br></a>
      </p>
      <img src="../assets/icons/5364.gif" alt="круто" class="mx-auto block" />
    </div>
  </XpWindow>
</template>
