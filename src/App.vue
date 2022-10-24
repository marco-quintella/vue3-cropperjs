<script setup lang="ts">
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
import { ref } from 'vue'
import Cropper from './components/Cropper.vue'

const resultImage = ref()
const selectedImage = ref()

const onSelectedFile = (event: Event) => {
  const file = (event.target as HTMLInputElement).files?.[0]
  if (file) {
    selectedImage.value = URL.createObjectURL(file)
  }
}
</script>

<template>
  <Cropper v-if="selectedImage" :src="selectedImage" @result="resultImage = $event" :aspect-ratio="16/9" />
  <img v-if="resultImage" :src="resultImage" />
  <input ref="fileSelectorRef" type="file" accept="image/*" @change="onSelectedFile" />
</template>
