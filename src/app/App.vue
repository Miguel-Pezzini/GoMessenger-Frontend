<script setup lang="ts">
import { ref, onMounted, useTemplateRef } from 'vue';
import { LogOut, MessageCirclePlus } from 'lucide-vue-next';
import ChatSidebar, { type Contact } from './components/ChatSidebar.vue';
import ChatHeader from './components/ChatHeader.vue';
import ChatMessages from './components/ChatMessages.vue';
import MessageInput from './components/MessageInput.vue';
import AuthCard from './components/AuthCard.vue';
import type { Message } from './components/MessageBubble.vue';

const API_BASE_URL = 'http://localhost:8080';

type AuthMode = 'login' | 'register';

type AuthCardExposed = {
  setError: (message: string) => void;
  setSuccess: (message: string) => void;
};

const authMode = ref<AuthMode>('login');
const isAuthenticated = ref(false);
const currentUser = ref('');
const authCardRef = useTemplateRef<AuthCardExposed>('authCardRef');

const mockContacts: Contact[] = [
  {
    id: '1',
    name: 'Ana Silva',
    avatar:
      'https://images.unsplash.com/photo-1649589244330-09ca58e4fa64?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&ixid=M3w3Nzg4Nzd8MHwxfHNlYXJjaHwxfHxwcm9mZXNzaW9uYWwlMjB3b21hbiUyMHBvcnRyYWl0fGVufDF8fHx8MTc3MjE4NzE5Mnww&ixlib=rb-4.1.0&q=80&w=1080',
    lastMessage: 'Ótimo! Vejo você amanhã então 😊',
    timestamp: '14:32',
    unreadCount: 3,
    online: true,
  },
  {
    id: '2',
    name: 'Carlos Mendes',
    avatar:
      'https://images.unsplash.com/photo-1554765345-6ad6a5417cde?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&ixid=M3w3Nzg4Nzd8MHwxfHNlYXJjaHwxfHxwcm9mZXNzaW9uYWwlMjBtYW4lMjBwb3J0cmFpdHxlbnwxfHx8fDE3NzIxODY2NTd8MA&ixlib=rb-4.1.0&q=80&w=1080',
    lastMessage: 'Você recebeu o relatório que enviei?',
    timestamp: '13:45',
    unreadCount: 0,
    online: false,
  },
  {
    id: '3',
    name: 'Mariana Costa',
    avatar:
      'https://images.unsplash.com/photo-1525786210598-d527194d3e9a?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&ixid=M3w3Nzg4Nzd8MHwxfHNlYXJjaHwxfHx5b3VuZyUyMHdvbWFuJTIwc21pbGluZ3xlbnwxfHx8fDE3NzIxNjU1MDd8MA&ixlib=rb-4.1.0&q=80&w=1080',
    lastMessage: 'Perfeito, obrigada!',
    timestamp: '12:20',
    unreadCount: 0,
    online: true,
  },
  {
    id: '4',
    name: 'Pedro Santos',
    avatar:
      'https://images.unsplash.com/photo-1723538201695-2427f49cdcde?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&ixid=M3w3Nzg4Nzd8MHwxfHNlYXJjaHwxfHxidXNpbmVzcyUyMG1hbiUyMGNhc3VhbHxlbnwxfHx8fDE3NzIyMzIzNjh8MA&ixlib=rb-4.1.0&q=80&w=1080',
    lastMessage: 'Vamos marcar uma reunião para discutir o projeto',
    timestamp: '11:05',
    unreadCount: 1,
    online: false,
  },
  {
    id: '5',
    name: 'Julia Oliveira',
    avatar:
      'https://images.unsplash.com/photo-1689600944138-da3b150d9cb8?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&ixid=M3w3Nzg4Nzd8MHwxfHNlYXJjaHwxfHx3b21hbiUyMHByb2Zlc3Npb25hbCUyMGhlYWRzaG90fGVufDF8fHx8MTc3MjE2NjI2NHww&ixlib=rb-4.1.0&q=80&w=1080',
    lastMessage: 'Conseguiu revisar os documentos?',
    timestamp: 'Ontem',
    unreadCount: 0,
    online: true,
  },
];

const initialMessages: Record<string, Message[]> = {
  '1': [
    { id: '1', text: 'Oi Ana! Tudo bem?', timestamp: '14:20', isMine: true },
    { id: '2', text: 'Oi! Tudo ótimo, e você?', timestamp: '14:21', isMine: false },
    {
      id: '3',
      text: 'Tudo bem também! Queria confirmar nossa reunião de amanhã',
      timestamp: '14:25',
      isMine: true,
    },
    { id: '4', text: 'Ah sim, está confirmada! Às 10h da manhã 😊', timestamp: '14:28', isMine: false },
    { id: '5', text: 'Perfeito! Vou preparar a apresentação hoje mesmo', timestamp: '14:30', isMine: true },
    { id: '6', text: 'Ótimo! Vejo você amanhã então 😊', timestamp: '14:32', isMine: false },
  ],
  '2': [
    { id: '1', text: 'Bom dia, Carlos!', timestamp: '09:15', isMine: true },
    { id: '2', text: 'Bom dia! Como vai o projeto?', timestamp: '09:20', isMine: false },
    { id: '3', text: 'Está indo bem. Terminei a primeira fase ontem', timestamp: '09:25', isMine: true },
    { id: '4', text: 'Que ótimo! Vou enviar o relatório para você', timestamp: '13:40', isMine: false },
    { id: '5', text: 'Você recebeu o relatório que enviei?', timestamp: '13:45', isMine: false },
  ],
  '3': [
    { id: '1', text: 'Mariana, você pode me ajudar com aquela dúvida?', timestamp: '12:10', isMine: true },
    { id: '2', text: 'Claro! Qual é a dúvida?', timestamp: '12:12', isMine: false },
    {
      id: '3',
      text: 'É sobre o formulário de cadastro. Qual campo devemos adicionar?',
      timestamp: '12:15',
      isMine: true,
    },
    { id: '4', text: 'Perfeito, obrigada!', timestamp: '12:20', isMine: false },
  ],
  '4': [
    { id: '1', text: 'Pedro, precisamos conversar sobre o cronograma', timestamp: '10:30', isMine: true },
    { id: '2', text: 'Vamos marcar uma reunião para discutir o projeto', timestamp: '11:05', isMine: false },
  ],
  '5': [
    { id: '1', text: 'Julia, enviei os documentos por email', timestamp: 'Ontem', isMine: true },
    { id: '2', text: 'Conseguiu revisar os documentos?', timestamp: 'Ontem', isMine: false },
  ],
};

const selectedContactId = ref<string>('1');
const messages = ref<Record<string, Message[]>>(initialMessages);
const selectedContact = ref<Contact | undefined>(mockContacts.find((c) => c.id === selectedContactId.value));
const currentMessages = ref<Message[]>(messages.value[selectedContactId.value] || []);

onMounted(() => {
  const token = localStorage.getItem('gomessenger_token');
  const username = localStorage.getItem('gomessenger_username');

  if (token && username) {
    isAuthenticated.value = true;
    currentUser.value = username;
  }
});

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
    timestamp: new Date().toLocaleTimeString('pt-BR', { hour: '2-digit', minute: '2-digit' }),
    isMine: true,
  };

  if (!messages.value[selectedContactId.value]) {
    messages.value[selectedContactId.value] = [];
  }

  messages.value[selectedContactId.value].push(newMessage);
  currentMessages.value = [...messages.value[selectedContactId.value]];
};

const getStatus = (contact: Contact) => (contact.online ? 'online' : 'offline');

const handleAuthSubmit = async (payload: { username: string; password: string; mode: AuthMode }) => {
  const endpoint = payload.mode === 'login' ? '/auth/login' : '/auth/register';

  const response = await fetch(`${API_BASE_URL}${endpoint}`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      username: payload.username,
      password: payload.password,
    }),
  });

  const data = await response.json().catch(() => ({}));

  if (!response.ok) {
    const message = data?.message || data?.error || 'Authentication failed. Please try again.';
    authCardRef.value?.setError(message);
    return;
  }

  if (payload.mode === 'register') {
    authCardRef.value?.setSuccess('Registration successful! You can sign in now.');
    authMode.value = 'login';
    return;
  }

  const token = data?.token || data?.accessToken || data?.jwt || 'session-authenticated';
  localStorage.setItem('gomessenger_token', token);
  localStorage.setItem('gomessenger_username', payload.username);
  currentUser.value = payload.username;
  isAuthenticated.value = true;
};

const toggleAuthMode = () => {
  authMode.value = authMode.value === 'login' ? 'register' : 'login';
};

const handleLogout = () => {
  localStorage.removeItem('gomessenger_token');
  localStorage.removeItem('gomessenger_username');
  currentUser.value = '';
  isAuthenticated.value = false;
  authMode.value = 'login';
};
</script>

<template>
  <main>
    <div
      v-if="!isAuthenticated"
      class="flex min-h-screen items-center justify-center bg-gradient-to-br from-slate-900 via-indigo-950 to-violet-900 px-4 py-8"
    >
      <div class="absolute inset-0 bg-[radial-gradient(circle_at_20%_20%,rgba(129,140,248,.2),transparent_35%),radial-gradient(circle_at_80%_0%,rgba(14,165,233,.18),transparent_25%)]" />
      <AuthCard ref="authCardRef" :mode="authMode" @submit="handleAuthSubmit" @toggleMode="toggleAuthMode" />
    </div>

    <div v-else class="relative mx-auto flex h-screen max-w-[1440px] bg-gradient-to-br from-indigo-50 via-purple-50 to-pink-50 shadow-2xl">
      <button
        type="button"
        class="absolute right-3 top-3 z-10 inline-flex items-center gap-1 rounded-full bg-white/90 px-3 py-2 text-xs font-semibold text-slate-700 shadow-md transition hover:bg-white"
        @click="handleLogout"
      >
        <LogOut :size="14" />
        Logout {{ currentUser }}
      </button>

      <ChatSidebar :contacts="mockContacts" :selectedContactId="selectedContactId" @selectContact="handleSelectContact" />

      <div v-if="selectedContact" class="flex flex-1 flex-col">
        <ChatHeader
          :name="selectedContact.name"
          :avatar="selectedContact.avatar"
          :status="getStatus(selectedContact)"
          :online="selectedContact.online"
        />
        <ChatMessages :messages="currentMessages" />
        <MessageInput @sendMessage="handleSendMessage" />
      </div>

      <div v-else class="flex flex-1 items-center justify-center">
        <div class="text-center">
          <div class="mx-auto mb-4 flex size-16 items-center justify-center rounded-full bg-indigo-100">
            <MessageCirclePlus :size="32" class="text-indigo-600" />
          </div>
          <p class="font-medium text-gray-600">Selecione uma conversa para começar</p>
          <p class="mt-1 text-sm text-gray-400">Escolha um contato da lista ao lado</p>
        </div>
      </div>
    </div>
  </main>
</template>
