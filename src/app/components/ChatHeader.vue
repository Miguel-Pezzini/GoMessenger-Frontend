<script setup lang="ts">
import { Phone, Video, MoreVertical } from 'lucide-vue-next';

interface Props {
  name: string;
  avatar: string;
  status: string;
  online: boolean;
}

defineProps<Props>();
</script>

<template>
  <div class="h-16 border-b border-indigo-100 px-6 flex items-center justify-between bg-white shadow-sm">
    <div class="flex items-center gap-3">
      <div class="relative">
        <div class="size-10 rounded-full overflow-hidden ring-2 ring-indigo-100 bg-gray-200 flex items-center justify-center">
          <img v-if="avatar" :src="avatar" :alt="name" class="w-full h-full object-cover" />
          <span v-else class="text-sm font-medium">{{ name.slice(0, 2).toUpperCase() }}</span>
        </div>
        <span v-if="online" class="absolute bottom-0 right-0 size-3 bg-emerald-500 border-2 border-white rounded-full" />
      </div>
      <div>
        <h2 class="font-semibold text-gray-900">{{ name }}</h2>
        <p class="text-sm text-indigo-600">{{ status }}</p>
      </div>
    </div>

    <div class="flex items-center gap-2">
      <button class="inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors hover:bg-indigo-50 h-10 w-10">
        <Phone :size="20" class="text-indigo-600" />
      </button>
      <button class="inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors hover:bg-indigo-50 h-10 w-10">
        <Video :size="20" class="text-indigo-600" />
      </button>
      <button class="inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors hover:bg-indigo-50 h-10 w-10">
        <MoreVertical :size="20" class="text-indigo-600" />
      </button>
    </div>
  </div>
</template>
