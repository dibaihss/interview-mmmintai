<script setup lang="ts">
import { ref } from 'vue'
import type { GalleryItem } from '@/lib/gallery-item.d.ts'
import DropField from '@/components/DropField.vue'
import ImageGallery from '@/components/ImageGallery.vue'

const galleryItems = ref<GalleryItem[]>([])
const acceptedFileTypes = '.jpg, .jpeg, .png, .webp'

function enumerate(fileList: FileList) {
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
        title: file.name
      }
      galleryItems.value.push(newItem)
      console.log('Image loaded:',galleryItems.value)
    }
    img.src = objectUrl
  }
}

const items = ref<GalleryItem[]>([
  {
    src: 'https://www.schadensmeldung.digital/images/fuhrparkmanagerin.webp',
    thumbnail: 'https://www.schadensmeldung.digital/images/fuhrparkmanagerin.webp',
    w: 0,
    h: 0,
    title: 'Will be used for caption',
  },
  {
    src: 'https://www.schadensmeldung.digital/images/logistik.webp',
    thumbnail: 'https://www.schadensmeldung.digital/images/logistik.webp',
    w: 0,
    h: 0,
  },
  {
    src: 'https://www.schadensmeldung.digital/images/werkstatt.webp',
    thumbnail: 'https://www.schadensmeldung.digital/images/werkstatt.webp',
    w: 0,
    h: 0,
  },
])
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
          <drop-field @drop="enumerate"></drop-field>
        </v-card-text>
      </v-card>
    </v-col>

    <v-col cols="12">
      <v-card>
        <v-card-title>Gallery</v-card-title>
        <v-card-text>
          <image-gallery :items="items"></image-gallery>
        </v-card-text>
      </v-card>
    </v-col>
  </v-row>
</template>
