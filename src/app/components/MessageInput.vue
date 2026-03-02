<script setup lang="ts">
import { ref } from 'vue';
import { Paperclip, Smile, Send } from 'lucide-vue-next';

const emit = defineEmits<{
  sendMessage: [text: string];
}>();

const message = ref('');

const handleSend = () => {
  if (message.value.trim()) {
    emit('sendMessage', message.value);
    message.value = '';
  }
};

const handleKeyDown = (e: KeyboardEvent) => {
  if (e.key === 'Enter' && !e.shiftKey) {
    e.preventDefault();
    handleSend();
  }
};
</script>

<template>
  <div class="border-t border-indigo-100 p-4 bg-white shadow-lg">
    <div class="flex items-end gap-2">
      <button class="shrink-0 inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors hover:bg-indigo-50 h-10 w-10">
        <Paperclip :size="20" class="text-indigo-600" />
      </button>
      
      <button class="shrink-0 inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors hover:bg-indigo-50 h-10 w-10">
        <Smile :size="20" class="text-indigo-600" />
      </button>

      <textarea
        v-model="message"
        @keydown="handleKeyDown"
        placeholder="Digite sua mensagem..."
        class="flex-1 resize-none min-h-[44px] max-h-32 rounded-xl border border-indigo-200 bg-white px-3 py-2 text-sm focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-indigo-400"
        rows="1"
      />

      <button
        @click="handleSend"
        :disabled="!message.trim()"
        class="shrink-0 inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors disabled:pointer-events-none disabled:opacity-50 bg-gradient-to-r from-indigo-600 to-purple-600 hover:from-indigo-700 hover:to-purple-700 text-white shadow-md h-10 px-4"
      >
        <Send :size="20" />
      </button>
    </div>
    <p class="text-xs text-indigo-400 mt-2 ml-20">
      Enter para enviar • Shift + Enter para quebrar linha
    </p>
  </div>
</template>
