<script setup lang="ts">
import { Search, MessageCirclePlus } from 'lucide-vue-next';

export interface Contact {
  id: string;
  name: string;
  avatar: string;
  lastMessage: string;
  timestamp: string;
  unreadCount: number;
  online: boolean;
}

interface Props {
  contacts: Contact[];
  selectedContactId: string | null;
}

defineProps<Props>();
const emit = defineEmits<{
  selectContact: [id: string];
}>();
</script>

<template>
  <div class="w-80 border-r border-gray-200 flex flex-col h-screen bg-gradient-to-b from-indigo-50 to-white">
    <!-- Header -->
    <div class="p-4 border-b border-indigo-100 bg-white/50 backdrop-blur-sm">
      <button class="w-full mb-3 inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 bg-indigo-600 hover:bg-indigo-700 text-white h-10 px-4 py-2">
        <MessageCirclePlus :size="16" class="mr-2" />
        Nova conversa
      </button>
      
      <div class="relative">
        <Search :size="16" class="absolute left-3 top-1/2 -translate-y-1/2 text-indigo-400" />
        <input
          type="text"
          placeholder="Pesquisar conversa"
          class="flex h-10 w-full rounded-md border border-indigo-200 bg-white px-3 py-2 pl-9 text-sm focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-indigo-400 focus-visible:ring-offset-0"
        />
      </div>
    </div>

    <!-- Contact List -->
    <div class="flex-1 overflow-y-auto">
      <div class="divide-y divide-indigo-50">
        <button
          v-for="contact in contacts"
          :key="contact.id"
          @click="emit('selectContact', contact.id)"
          :class="[
            'w-full p-4 flex items-start gap-3 hover:bg-indigo-50/50 transition-all text-left',
            selectedContactId === contact.id ? 'bg-indigo-100/70 border-l-4 border-indigo-600' : ''
          ]"
        >
          <div class="relative">
            <div class="size-10 rounded-full overflow-hidden ring-2 ring-white bg-gray-200 flex items-center justify-center">
              <img v-if="contact.avatar" :src="contact.avatar" :alt="contact.name" class="w-full h-full object-cover" />
              <span v-else class="text-sm font-medium">{{ contact.name.slice(0, 2).toUpperCase() }}</span>
            </div>
            <span v-if="contact.online" class="absolute bottom-0 right-0 size-3 bg-emerald-500 border-2 border-white rounded-full" />
          </div>

          <div class="flex-1 min-w-0">
            <div class="flex items-center justify-between mb-1">
              <h3 class="font-medium text-sm truncate text-gray-900">{{ contact.name }}</h3>
              <span class="text-xs text-indigo-500 font-medium">{{ contact.timestamp }}</span>
            </div>
            <div class="flex items-center justify-between gap-2">
              <p class="text-sm text-gray-600 truncate flex-1">{{ contact.lastMessage }}</p>
              <span
                v-if="contact.unreadCount > 0"
                class="bg-indigo-600 hover:bg-indigo-700 text-white size-5 flex items-center justify-center rounded-full text-[10px] font-medium"
              >
                {{ contact.unreadCount }}
              </span>
            </div>
          </div>
        </button>
      </div>
    </div>
  </div>
</template>
