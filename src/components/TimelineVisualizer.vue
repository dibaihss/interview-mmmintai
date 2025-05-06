<script setup lang="ts">
import { ref, onMounted } from 'vue'
import type { ProcessItem } from '@/lib/process-item'

const props = defineProps<{
  processItems: ProcessItem[]
  activeStepIndex?: number
}>()

const emit = defineEmits(['update:activeStepIndex'])

const localActiveStepIndex = ref(props.activeStepIndex !== undefined ? props.activeStepIndex : -1)

const getStepColor = (index: number): string => {
  if (localActiveStepIndex.value === -1) {
    return 'primary'
  }

  if (index === localActiveStepIndex.value) {
    return 'primary'
  } else if (index < localActiveStepIndex.value) {
    return 'success'
  } else {
    return 'grey'
  }
}

onMounted(() => {
  setActiveStep()
})

// Format timestamp to display date and time in German format
const formatDate = (timestamp: string): string => {
  return new Date(timestamp).toLocaleString('de-DE', {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
  })
}

const setActiveStep = () => {
  const index = localActiveStepIndex.value
  emit('update:activeStepIndex', index)
}
</script>

<template>
  <div class="timeline-container">
    <div class="process-timeline">
      <v-timeline
        align="start"
        side="end"
        direction="vertical"
        density="comfortable"
        line-thickness="3"
      >
        <v-timeline-item
          v-for="(item, index) in processItems"
          :key="item.type"
          :size="index === localActiveStepIndex ? 'large' : 'small'"
          :class="{
            'active-item': index === localActiveStepIndex,
            'completed-item': index < localActiveStepIndex,
          }"
        >
          <template v-slot:opposite>
            <div class="date">{{ formatDate(item.timestamp) }}</div>
          </template>
          <v-card
            class="timeline-card"
            :class="{
              'active-card': index === localActiveStepIndex,
              'completed-card': index < localActiveStepIndex,
              'future-card': index > localActiveStepIndex,
            }"
            :color="index === localActiveStepIndex ? 'primary' : ''"
            :variant="index === localActiveStepIndex ? 'flat' : 'outlined'"
            elevation="4"
          >
            <v-card-title class="text-wrap">
              <div class="d-flex align-center">
                <span class="step-number mr-2">{{ index + 1 }}</span>
                {{ item.title }}
                <v-chip class="ml-2" :color="getStepColor(index)" size="small" label>
                  {{ item.status }}
                </v-chip>
              </div>
            </v-card-title>

            <v-card-subtitle class="pb-0">
              <v-icon icon="mdi-account" size="small" class="mr-1"></v-icon>
              {{ item.contact }}
              <span class="mx-1">•</span>
              <v-icon icon="mdi-map-marker" size="small" class="mr-1"></v-icon>
              {{ item.location }}
            </v-card-subtitle>
          </v-card>
        </v-timeline-item>
      </v-timeline>
    </div>
  </div>
</template>

<style scoped>
.timeline-container {
  max-width: 100%;
  padding: 16px 0;
}

.process-timeline {
  position: relative;
}

.timeline-card {
  margin-bottom: 12px;
  transition: all 0.3s ease;
  border-left: 4px solid transparent;
}

.active-card {
  box-shadow: 0 4px 12px rgba(25, 118, 210, 0.35) !important;
  border-left: 4px solid var(--v-primary-base);
}

.completed-card {
  border-left: 4px solid #4caf50;
}

.future-card {
  opacity: 0.85;
}

.active-item :deep(.v-timeline-divider__dot) {
  box-shadow: 0 0 0 4px rgba(25, 118, 210, 0.2);
}

.completed-item :deep(.v-timeline-divider__dot) {
  box-shadow: 0 0 0 2px rgba(76, 175, 80, 0.15);
}

.step-number {
  display: flex;
  align-items: center;
  justify-content: center;
  min-width: 24px;
  height: 24px;
  border-radius: 50%;
  background-color: rgba(0, 0, 0, 0.1);
  font-size: 0.8rem;
  font-weight: bold;
  color: rgba(0, 0, 0, 0.7);
}

.active-card .step-number {
  background-color: white;
  color: var(--v-primary-base);
}
.date {
  color: rgba(0, 0, 0, 0.87);
  font-weight: 500;
}
</style>
