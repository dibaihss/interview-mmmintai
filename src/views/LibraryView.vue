<script setup lang="ts">
import { ref } from 'vue'
import type { GalleryItem } from '@/lib/gallery-item.d.ts'
import DropField from '@/components/DropField.vue'
import ImageGallery from '@/components/ImageGallery.vue'

const galleryItems = ref<GalleryItem[]>([])
const acceptedFileTypes = '.jpg, .jpeg, .png, .webp'

function handleFilesDrop(fileList: FileList) {
  for (let i = 0; i < fileList.length; i++) {
    const file = fileList[i]
    const objectUrl = URL.createObjectURL(file)

    const img = new Image()
    img.onload = () => {
      const newItem: GalleryItem = {
        src: objectUrl,
        thumbnail: objectUrl,
        w: img.width,
        h: img.height,
        title: file.name,
      }
      galleryItems.value.push(newItem)
      console.log('Image loaded:', galleryItems.value)
    }
    img.src = objectUrl
  }
}
</script>

<template>
  <v-row>
    <v-col cols="12">
      <h1>Library</h1>
    </v-col>

    <v-col cols="12">
      <v-card>
        <v-card-title>Drop Field</v-card-title>
        <v-card-text>
          <DropField :accept="acceptedFileTypes" @drop="handleFilesDrop" />
        </v-card-text>
      </v-card>
    </v-col>

    <v-col cols="12">
      <v-card>
        <v-card-title>Gallery</v-card-title>
        <v-card-text>
          <div v-if="galleryItems.length === 0" class="text-center pa-4">
            <p>No images uploaded yet. Drop images in the field above.</p>
          </div>
          <ImageGallery v-else :items="galleryItems" />
        </v-card-text>
      </v-card>
    </v-col>
  </v-row>
</template>
