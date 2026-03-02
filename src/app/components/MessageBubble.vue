<script setup lang="ts">
export interface Message {
  id: string;
  text?: string;
  image?: string;
  timestamp: string;
  isMine: boolean;
}

interface Props {
  message: Message;
}

defineProps<Props>();
</script>

<template>
  <div :class="['mb-4', message.isMine ? 'justify-end' : 'justify-start']">
    <div :class="['flex flex-col', message.isMine ? 'items-end' : 'items-start']">
      <div
        v-if="message.image"
        :class="[
          'rounded-2xl overflow-hidden mb-1 p-1.5',
          message.isMine ? 'bg-gradient-to-br from-indigo-500 to-purple-600' : 'bg-white shadow-md'
        ]"
      >
        <img :src="message.image" alt="Imagem enviada" class="rounded-xl max-w-full h-auto" />
      </div>
      
      <div
        v-if="message.text"
        :class="[
          'rounded-2xl px-4 py-3 shadow-sm',
          message.isMine
            ? 'bg-gradient-to-br from-indigo-600 to-purple-600 text-white rounded-br-sm'
            : 'bg-white text-gray-900 rounded-bl-sm border border-gray-100'
        ]"
      >
        <p class="whitespace-pre-wrap break-words">{{ message.text }}</p>
      </div>
      
      <span class="text-xs text-gray-500 mt-1.5 px-1">
        {{ message.timestamp }}
      </span>
    </div>
  </div>
</template>
