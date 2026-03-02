<script setup lang="ts">
import { ref } from 'vue';
import { MessageCirclePlus } from 'lucide-vue-next';
import ChatSidebar, { type Contact } from './components/ChatSidebar.vue';
import ChatHeader from './components/ChatHeader.vue';
import ChatMessages from './components/ChatMessages.vue';
import MessageInput from './components/MessageInput.vue';
import type { Message } from './components/MessageBubble.vue';

// Mock data para conversas
const mockContacts: Contact[] = [
  {
    id: "1",
    name: "Ana Silva",
    avatar: "https://images.unsplash.com/photo-1649589244330-09ca58e4fa64?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&ixid=M3w3Nzg4Nzd8MHwxfHNlYXJjaHwxfHxwcm9mZXNzaW9uYWwlMjB3b21hbiUyMHBvcnRyYWl0fGVufDF8fHx8MTc3MjE4NzE5Mnww&ixlib=rb-4.1.0&q=80&w=1080",
    lastMessage: "Ótimo! Vejo você amanhã então 😊",
    timestamp: "14:32",
    unreadCount: 3,
    online: true,
  },
  {
    id: "2",
    name: "Carlos Mendes",
    avatar: "https://images.unsplash.com/photo-1554765345-6ad6a5417cde?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&ixid=M3w3Nzg4Nzd8MHwxfHNlYXJjaHwxfHxwcm9mZXNzaW9uYWwlMjBtYW4lMjBwb3J0cmFpdHxlbnwxfHx8fDE3NzIxODY2NTd8MA&ixlib=rb-4.1.0&q=80&w=1080",
    lastMessage: "Você recebeu o relatório que enviei?",
    timestamp: "13:45",
    unreadCount: 0,
    online: false,
  },
  {
    id: "3",
    name: "Mariana Costa",
    avatar: "https://images.unsplash.com/photo-1525786210598-d527194d3e9a?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&ixid=M3w3Nzg4Nzd8MHwxfHNlYXJjaHwxfHx5b3VuZyUyMHdvbWFuJTIwc21pbGluZ3xlbnwxfHx8fDE3NzIxNjU1MDd8MA&ixlib=rb-4.1.0&q=80&w=1080",
    lastMessage: "Perfeito, obrigada!",
    timestamp: "12:20",
    unreadCount: 0,
    online: true,
  },
  {
    id: "4",
    name: "Pedro Santos",
    avatar: "https://images.unsplash.com/photo-1723538201695-2427f49cdcde?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&ixid=M3w3Nzg4Nzd8MHwxfHNlYXJjaHwxfHxidXNpbmVzcyUyMG1hbiUyMGNhc3VhbHxlbnwxfHx8fDE3NzIyMzIzNjh8MA&ixlib=rb-4.1.0&q=80&w=1080",
    lastMessage: "Vamos marcar uma reunião para discutir o projeto",
    timestamp: "11:05",
    unreadCount: 1,
    online: false,
  },
  {
    id: "5",
    name: "Julia Oliveira",
    avatar: "https://images.unsplash.com/photo-1689600944138-da3b150d9cb8?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&ixid=M3w3Nzg4Nzd8MHwxfHNlYXJjaHwxfHx3b21hbiUyMHByb2Zlc3Npb25hbCUyMGhlYWRzaG90fGVufDF8fHx8MTc3MjE2NjI2NHww&ixlib=rb-4.1.0&q=80&w=1080",
    lastMessage: "Conseguiu revisar os documentos?",
    timestamp: "Ontem",
    unreadCount: 0,
    online: true,
  },
];

// Mock data para mensagens
const initialMessages: Record<string, Message[]> = {
  "1": [
    { id: "1", text: "Oi Ana! Tudo bem?", timestamp: "14:20", isMine: true },
    { id: "2", text: "Oi! Tudo ótimo, e você?", timestamp: "14:21", isMine: false },
    { id: "3", text: "Tudo bem também! Queria confirmar nossa reunião de amanhã", timestamp: "14:25", isMine: true },
    { id: "4", text: "Ah sim, está confirmada! Às 10h da manhã 😊", timestamp: "14:28", isMine: false },
    { id: "5", text: "Perfeito! Vou preparar a apresentação hoje mesmo", timestamp: "14:30", isMine: true },
    { id: "6", text: "Ótimo! Vejo você amanhã então 😊", timestamp: "14:32", isMine: false },
  ],
  "2": [
    { id: "1", text: "Bom dia, Carlos!", timestamp: "09:15", isMine: true },
    { id: "2", text: "Bom dia! Como vai o projeto?", timestamp: "09:20", isMine: false },
    { id: "3", text: "Está indo bem. Terminei a primeira fase ontem", timestamp: "09:25", isMine: true },
    { id: "4", text: "Que ótimo! Vou enviar o relatório para você", timestamp: "13:40", isMine: false },
    { id: "5", text: "Você recebeu o relatório que enviei?", timestamp: "13:45", isMine: false },
  ],
  "3": [
    { id: "1", text: "Mariana, você pode me ajudar com aquela dúvida?", timestamp: "12:10", isMine: true },
    { id: "2", text: "Claro! Qual é a dúvida?", timestamp: "12:12", isMine: false },
    { id: "3", text: "É sobre o formulário de cadastro. Qual campo devemos adicionar?", timestamp: "12:15", isMine: true },
    { id: "4", text: "Perfeito, obrigada!", timestamp: "12:20", isMine: false },
  ],
  "4": [
    { id: "1", text: "Pedro, precisamos conversar sobre o cronograma", timestamp: "10:30", isMine: true },
    { id: "2", text: "Vamos marcar uma reunião para discutir o projeto", timestamp: "11:05", isMine: false },
  ],
  "5": [
    { id: "1", text: "Julia, enviei os documentos por email", timestamp: "Ontem", isMine: true },
    { id: "2", text: "Conseguiu revisar os documentos?", timestamp: "Ontem", isMine: false },
  ],
};

const selectedContactId = ref<string>("1");
const messages = ref<Record<string, Message[]>>(initialMessages);

const selectedContact = ref<Contact | undefined>(
  mockContacts.find((c) => c.id === selectedContactId.value)
);

const currentMessages = ref<Message[]>(messages.value[selectedContactId.value] || []);

const handleSelectContact = (id: string) => {
  selectedContactId.value = id;
  selectedContact.value = mockContacts.find((c) => c.id === id);
  currentMessages.value = messages.value[id] || [];
};

const handleSendMessage = (text: string) => {
  if (!selectedContactId.value) return;

  const newMessage: Message = {
    id: Date.now().toString(),
    text,
    timestamp: new Date().toLocaleTimeString("pt-BR", { hour: "2-digit", minute: "2-digit" }),
    isMine: true,
  };

  if (!messages.value[selectedContactId.value]) {
    messages.value[selectedContactId.value] = [];
  }
  
  messages.value[selectedContactId.value].push(newMessage);
  currentMessages.value = [...messages.value[selectedContactId.value]];
};

const getStatus = (contact: Contact) => {
  return contact.online ? "online" : "offline";
};
</script>

<template>
  <div class="flex h-screen bg-gradient-to-br from-indigo-50 via-purple-50 to-pink-50 max-w-[1440px] mx-auto shadow-2xl">
    <ChatSidebar
      :contacts="mockContacts"
      :selectedContactId="selectedContactId"
      @selectContact="handleSelectContact"
    />

    <div v-if="selectedContact" class="flex-1 flex flex-col">
      <ChatHeader
        :name="selectedContact.name"
        :avatar="selectedContact.avatar"
        :status="getStatus(selectedContact)"
        :online="selectedContact.online"
      />
      <ChatMessages :messages="currentMessages" />
      <MessageInput @sendMessage="handleSendMessage" />
    </div>

    <div v-else class="flex-1 flex items-center justify-center">
      <div class="text-center">
        <div class="size-16 mx-auto mb-4 rounded-full bg-indigo-100 flex items-center justify-center">
          <MessageCirclePlus :size="32" class="text-indigo-600" />
        </div>
        <p class="text-gray-600 font-medium">Selecione uma conversa para começar</p>
        <p class="text-sm text-gray-400 mt-1">Escolha um contato da lista ao lado</p>
      </div>
    </div>
  </div>
</template>
