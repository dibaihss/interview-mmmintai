<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'
import type { GalleryItem } from '@/lib/gallery-item.d.ts'

const props = defineProps({ items: Array as () => GalleryItem[] })

const currentIndex = ref(0)
const rotationDegrees = ref(0)
const zoomLevel = ref(1)
const galleryModalOpen = ref(false)

const showNext = () => {
  if (!props.items?.length) return
  currentIndex.value = (currentIndex.value + 1) % props.items.length
  resetImageTransform()
}

const showPrevious = () => {
  if (!props.items?.length) return
  currentIndex.value = (currentIndex.value - 1 + props.items.length) % props.items.length
  resetImageTransform()
}

const rotateImage = () => {
  rotationDegrees.value = (rotationDegrees.value + 90) % 360
}

const zoomIn = () => {
  if (zoomLevel.value < 3) {
    zoomLevel.value += 0.25
  }
}

const zoomOut = () => {
  if (zoomLevel.value > 0.5) {
    zoomLevel.value -= 0.25
  }
}

const resetImageTransform = () => {
  rotationDegrees.value = 0
  zoomLevel.value = 1
}

const openGalleryModal = (index: number) => {
  currentIndex.value = index
  galleryModalOpen.value = true
  resetImageTransform()
}

const closeGalleryModal = () => {
  galleryModalOpen.value = false
}

// Keyboard navigation
const handleKeyDown = (e: KeyboardEvent) => {
  if (!galleryModalOpen.value) return

  switch (e.key) {
    case 'ArrowRight':
      showNext()
      break
    case 'ArrowLeft':
      showPrevious()
      break
    case 'r':
    case 'R':
      rotateImage()
      break
    case '+':
      zoomIn()
      break
    case '-':
      zoomOut()
      break
    case 'Escape':
      closeGalleryModal()
      break
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeyDown)
})

onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleKeyDown)
})
</script>

<template>
  <div class="image-gallery">
    <!-- Thumbnails grid -->
    <v-row v-if="props.items?.length">
      <v-col
        v-for="(item, index) in props.items"
        :key="index"
        cols="4"
        md="3"
        lg="2"
        class="thumbnail-container"
      >
        <v-hover v-slot="{ isHovering, props: hoverProps }">
          <v-card v-bind="hoverProps" class="thumbnail-card" @click="openGalleryModal(index)">
            <v-img
              :src="item.thumbnail"
              :alt="item.title || 'Image'"
              cover
              height="150px"
              class="thumbnail-image"
            >
              <v-fade-transition>
                <div
                  v-if="isHovering"
                  class="d-flex justify-center align-center fill-height thumbnail-overlay"
                >
                  <v-btn icon variant="text" color="white" size="small">
                    <v-icon>mdi-magnify-plus</v-icon>
                  </v-btn>
                </div>
              </v-fade-transition>
            </v-img>
            <v-card-text class="pa-2 text-caption text-truncate">
              {{ item.title || 'Image' }}
            </v-card-text>
          </v-card>
        </v-hover>
      </v-col>
    </v-row>

    <!-- Gallery Modal -->
    <v-dialog
      v-model="galleryModalOpen"
      fullscreen
      transition="dialog-bottom-transition"
      :scrim="false"
      content-class="gallery-modal"
    >
      <v-card class="gallery-modal-card">
        <v-toolbar color="primary" dark>
          <v-btn icon @click="closeGalleryModal">
            <v-icon>mdi-close</v-icon>
          </v-btn>
          <v-toolbar-title>{{
            props.items?.[currentIndex]?.title || 'Image Gallery'
          }}</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-toolbar-items>
            <v-btn icon @click="zoomOut">
              <v-icon>mdi-magnify-minus</v-icon>
            </v-btn>
            <v-btn icon @click="zoomIn">
              <v-icon>mdi-magnify-plus</v-icon>
            </v-btn>
            <v-btn icon @click="rotateImage">
              <v-icon>mdi-rotate-right</v-icon>
            </v-btn>
            <v-btn icon @click="resetImageTransform">
              <v-icon>mdi-restore</v-icon>
            </v-btn>
          </v-toolbar-items>
        </v-toolbar>

        <div class="gallery-content">
          <div class="navigation-buttons">
            <v-btn
              icon
              class="nav-button prev-button"
              @click="showPrevious"
              v-if="props.items?.length > 1"
            >
              <v-icon>mdi-chevron-left</v-icon>
            </v-btn>
            <v-btn
              icon
              class="nav-button next-button"
              @click="showNext"
              v-if="props.items?.length > 1"
            >
              <v-icon>mdi-chevron-right</v-icon>
            </v-btn>
          </div>

          <div class="image-container">
            <img
              v-if="props.items?.[currentIndex]"
              :src="props.items[currentIndex].src"
              :alt="props.items[currentIndex].title || 'Image'"
              class="gallery-image"
              :style="{
                transform: `rotate(${rotationDegrees}deg) scale(${zoomLevel})`,
                'max-width': rotationDegrees % 180 === 0 ? '100%' : '80vh',
              }"
            />
          </div>
        </div>

        <div class="image-info">
          <p v-if="props.items?.[currentIndex]">
            {{ currentIndex + 1 }} / {{ props.items?.length }} -
            {{ props.items[currentIndex].title || 'Image' }}
          </p>
          <p class="text-caption">Use arrow keys to navigate, R to rotate, + and - to zoom</p>
        </div>
      </v-card>
    </v-dialog>
  </div>
</template>

<style scoped>
.image-gallery {
  width: 100%;
}

.thumbnail-container {
  padding: 4px;
}

.thumbnail-card {
  cursor: pointer;
  transition: transform 0.2s;
}

.thumbnail-card:hover {
  transform: scale(1.05);
}

.thumbnail-overlay {
  background-color: rgba(0, 0, 0, 0.5);
}

.gallery-modal-card {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.gallery-content {
  flex: 1;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: rgba(0, 0, 0, 0.9);
  overflow: hidden;
}

.image-container {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  width: 100%;
}

.gallery-image {
  max-height: 85vh;
  object-fit: contain;
  transition: transform 0.3s ease;
}

.navigation-buttons {
  position: absolute;
  width: 100%;
  display: flex;
  justify-content: space-between;
  padding: 0 16px;
  z-index: 1;
}

.nav-button {
  background-color: rgba(0, 0, 0, 0.5) !important;
  color: white !important;
}

.image-info {
  padding: 8px 16px;
  background-color: rgba(0, 0, 0, 0.8);
  color: white;
  text-align: center;
}

@media (max-width: 600px) {
  .nav-button {
    transform: scale(0.8);
  }
}
</style>
