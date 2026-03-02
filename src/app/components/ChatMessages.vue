<script setup lang="ts">
import { ref, watch, nextTick } from 'vue';
import MessageBubble, { type Message } from './MessageBubble.vue';

interface Props {
  messages: Message[];
}

const props = defineProps<Props>();
const scrollRef = ref<HTMLDivElement | null>(null);

const scrollToBottom = () => {
  if (scrollRef.value) {
    scrollRef.value.scrollTop = scrollRef.value.scrollHeight;
  }
};

watch(() => props.messages, async () => {
  await nextTick();
  scrollToBottom();
}, { deep: true });
</script>

<template>
  <div class="flex-1 bg-gradient-to-b from-gray-50 to-indigo-50/30 overflow-y-auto" ref="scrollRef">
    <div class="p-6 flex flex-col min-h-full">
      <MessageBubble
        v-for="message in messages"
        :key="message.id"
        :message="message"
      />
    </div>
  </div>
</template>
