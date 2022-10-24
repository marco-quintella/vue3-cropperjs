<template>
  <div id="cropper-container" :style="containerStyle" :class="containerClass" :alt="alt">
    <img id="image" ref="imgRef" :src="src" :style="imgStyle" :class="imgClass" />
  </div>
  <button @click="crop">Crop</button>
</template>
<script setup lang="ts">
import Cropper from 'cropperjs'
import 'cropperjs/dist/cropper.css'
import type { PropType } from 'vue'
import { onBeforeUnmount, onMounted, ref, watch } from 'vue'

type ViewMode = Cropper.ViewMode

const props = defineProps({
  alt: String,
  aspectRatio: Number,
  containerClass: String || Object,
  containerStyle: String || Object,
  height: Number,
  imgClass: String || Object,
  imgStyle: String || Object,
  maxHeight: Number,
  maxWidth: Number,
  rounded: Boolean,
  src: {
    default: '',
    type: String
  },
  viewMode: {
    default: 3,
    type: (Object as PropType<ViewMode>)
  },
  width: Number
})

const emit = defineEmits<{
  (e: 'result', value: string): void
  (e: 'ready'): void
  (e: 'start'): void
  (e: 'move'): void
  (e: 'end'): void
  (e: 'crop'): void
  (e: 'zoom'): void
}>()

const imgRef = ref<HTMLImageElement>()
const cropperInstance = ref<Cropper>()
const resultImage = ref<string>()
const ready = ref(false)
const roundedClassIndex = ref<number>()


// Starting & Stopping Methods
const start = () => {
  if (ready.value || !imgRef.value) {
    return
  }
  if (props.rounded) {
    document.styleSheets[0].insertRule('.cropper-view-box, .cropper-face { border-radius: 50% }')
  }
  cropperInstance.value = new Cropper(imgRef.value, {
    aspectRatio: props.aspectRatio,
    viewMode: props.viewMode,
    ready: () => {
      ready.value = true
      emit('ready')
    },
    cropstart: () => emit('start'),
    cropmove: () => emit('move'),
    cropend: (e) => emit('end'),
    crop: (e) => emit('crop'),
    zoom: () => emit('zoom')
  })
}

const stop = () => {
  if (cropperInstance.value) {
    ready.value = false
    cropperInstance.value.destroy()
    cropperInstance.value = undefined
  }
}

// Cropping Methods
const crop = () => {
  if (!ready.value || !cropperInstance.value) {
    return
  }

  const options: Cropper.GetCroppedCanvasOptions = {}

  if (props.maxHeight) options.maxHeight = props.maxHeight
  if (props.maxWidth) options.maxWidth = props.maxWidth
  if (props.height) options.height = props.height
  if (props.width) options.width = props.width

  const croppedCanvas = cropperInstance.value.getCroppedCanvas(options)

  if (props.rounded) {
    const roundedCanvas = getRoundedCanvas(croppedCanvas)
    resultImage.value = roundedCanvas.toDataURL()
  } else {
    resultImage.value = croppedCanvas.toDataURL()
  }
  emit('result', resultImage.value)
}

const getRoundedCanvas = (sourceCanvas: HTMLCanvasElement) => {
  const canvas = document.createElement('canvas')
  const context = canvas.getContext('2d')
  const width = sourceCanvas.width
  const height = sourceCanvas.height

  canvas.width = width
  canvas.height = height

  if (context) {
    context.imageSmoothingEnabled = true
    context.drawImage(sourceCanvas, 0, 0, width, height)
    context.globalCompositeOperation = 'destination-in'
    context.beginPath()
    context.arc(width / 2, height / 2, width / 2, 0, Math.PI * 2)
    context.fill()
  }

  return canvas
}

// Mounting and Unmounting Cropper
onMounted(() => {
  start()
})

onBeforeUnmount(() => {
  stop()
})

// Updated image when changed after first choose
watch(() => props.src, () => {
  cropperInstance.value?.replace(props.src)
})

// Updates when rounded is removed, enabling to create a rounded/square button
// Rounded class index is used to remove the rounded class from the style sheet
const insertRoundedClass = () => {
  if (props.rounded && !roundedClassIndex.value) {
    roundedClassIndex.value = document.styleSheets[0].insertRule('.cropper-view-box, .cropper-face { border-radius: 50% }')
  }
}

const removeRoundedClass = () => {
  if (!props.rounded && roundedClassIndex.value) {
    document.styleSheets[0].deleteRule(roundedClassIndex.value)
  }
}

watch(() => props.rounded, () => {
  if (props.rounded) {
    insertRoundedClass()
  } else {
    removeRoundedClass()
  }
})
</script>
<style scoped>
img {
  max-width: 100%;
}
</style>
<style>
.cropper-view-box {
  outline: 0;
  box-shadow: 0 0 0 1px #39f;
}
</style>