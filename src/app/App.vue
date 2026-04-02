<script setup lang="ts">
import { computed, onMounted, ref, useTemplateRef, watch } from 'vue';
import { LogOut, MessageCirclePlus } from 'lucide-vue-next';
import ChatSidebar, { type Contact, type PendingRequest } from './components/ChatSidebar.vue';
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

type Friend = {
  id: string;
  userId: string;
  friendId: string;
  createdAt: string;
};

type FriendRequest = {
  id: string;
  senderId: string;
  receiverId: string;
  createdAt: string;
};

const authMode = ref<AuthMode>('login');
const isAuthenticated = ref(false);
const isFriendsLoading = ref(false);
const currentUser = ref('');
const currentUserId = ref('');
const actionError = ref('');
const actionSuccess = ref('');
const authCardRef = useTemplateRef<AuthCardExposed>('authCardRef');

let ws: WebSocket | null = null;

const connectWebSocket = (token: string) => {
  ws = new WebSocket(`ws://localhost:8080/ws?token=${encodeURIComponent(token)}`);

  ws.addEventListener('message', (event: MessageEvent) => {
    try {
      const msg = JSON.parse(event.data as string) as { type: string };
      if (
        msg.type === 'friend_request_received' ||
        msg.type === 'friend_request_accepted' ||
        msg.type === 'friend_request_declined' ||
        msg.type === 'friend_removed'
      ) {
        void refreshFriendsData();
      }
    } catch {
      // ignore non-JSON messages
    }
  });
};

const disconnectWebSocket = () => {
  ws?.close();
  ws = null;
};

const friends = ref<Friend[]>([]);
const pendingRequests = ref<FriendRequest[]>([]);
const selectedContactId = ref<string | null>(null);
const messages = ref<Record<string, Message[]>>({});

const contacts = computed<Contact[]>(() =>
  friends.value.map((friend) => ({
    id: friend.friendId,
    name: friend.friendId,
    avatar: '',
    lastMessage: 'Friend connection created in backend',
    timestamp: formatTimestamp(friend.createdAt),
    unreadCount: 0,
    online: false,
  }))
);

const selectedContact = computed<Contact | undefined>(() =>
  contacts.value.find((contact) => contact.id === selectedContactId.value)
);

const currentMessages = computed<Message[]>(() => {
  if (!selectedContactId.value) {
    return [];
  }

  return messages.value[selectedContactId.value] || [];
});

const sidebarPendingRequests = computed<PendingRequest[]>(() =>
  pendingRequests.value.map((request) => ({
    id: request.id,
    senderId: request.senderId,
    receiverId: request.receiverId,
    createdAt: formatTimestamp(request.createdAt),
  }))
);

watch(contacts, (nextContacts) => {
  if (selectedContactId.value && nextContacts.some((contact) => contact.id === selectedContactId.value)) {
    return;
  }

  selectedContactId.value = nextContacts[0]?.id ?? null;
});

onMounted(() => {
  void initializeSession();
});

const initializeSession = async () => {
  const token = localStorage.getItem('gomessenger_token');
  const username = localStorage.getItem('gomessenger_username');

  if (!token || !username) {
    return;
  }

  isAuthenticated.value = true;
  currentUser.value = username;
  currentUserId.value = parseJwtUserId(token);
  connectWebSocket(token);

  try {
    await refreshFriendsData();
  } catch (error) {
    actionError.value = getErrorMessage(error, 'Failed to load friends.');
  }
};

const parseJwtUserId = (token: string) => {
  const parts = token.split('.');
  if (parts.length < 2) {
    return '';
  }

  try {
    const normalized = parts[1].replace(/-/g, '+').replace(/_/g, '/');
    const padded = normalized.padEnd(normalized.length + ((4 - (normalized.length % 4)) % 4), '=');
    const payload = JSON.parse(atob(padded));
    return typeof payload.userId === 'string' ? payload.userId : '';
  } catch {
    return '';
  }
};

const formatTimestamp = (value: string) => {
  const date = new Date(value);
  if (Number.isNaN(date.getTime())) {
    return value;
  }

  return new Intl.DateTimeFormat('en-US', {
    month: 'short',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
  }).format(date);
};

const getErrorMessage = (error: unknown, fallback: string) => {
  return error instanceof Error && error.message ? error.message : fallback;
};

const authFetch = async <T>(path: string, init: RequestInit = {}): Promise<T> => {
  const token = localStorage.getItem('gomessenger_token');
  if (!token) {
    throw new Error('Missing session token. Please sign in again.');
  }

  const headers = new Headers(init.headers);
  headers.set('Authorization', `Bearer ${token}`);

  if (init.body && !headers.has('Content-Type')) {
    headers.set('Content-Type', 'application/json');
  }

  const response = await fetch(`${API_BASE_URL}${path}`, {
    ...init,
    headers,
  });

  const payload = response.status === 204 ? null : await response.json().catch(() => null);

  if (!response.ok) {
    const message =
      (payload &&
        typeof payload === 'object' &&
        ('error' in payload || 'message' in payload) &&
        String((payload as { error?: string; message?: string }).error || (payload as { message?: string }).message)) ||
      `Request failed with status ${response.status}`;

    if (response.status === 401) {
      handleLogout();
    }

    throw new Error(message);
  }

  return payload as T;
};

const refreshFriendsData = async () => {
  isFriendsLoading.value = true;

  try {
    const [friendsResponse, pendingResponse] = await Promise.all([
      authFetch<Friend[]>('/friends'),
      authFetch<FriendRequest[]>('/friends/requests/pending'),
    ]);

    friends.value = friendsResponse;
    pendingRequests.value = pendingResponse;
  } finally {
    isFriendsLoading.value = false;
  }
};

const handleSelectContact = (id: string) => {
  selectedContactId.value = id;
};

const handleAuthSubmit = async (payload: { username: string; password: string; mode: AuthMode }) => {
  const endpoint = payload.mode === 'login' ? '/auth/login' : '/auth/register';

  try {
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

    const token = data?.token || data?.accessToken || data?.jwt || '';
    if (!token) {
      authCardRef.value?.setError('Login succeeded but no token was returned.');
      return;
    }

    localStorage.setItem('gomessenger_token', token);
    localStorage.setItem('gomessenger_username', payload.username);
    currentUser.value = payload.username;
    currentUserId.value = parseJwtUserId(token);
    isAuthenticated.value = true;
    actionError.value = '';
    actionSuccess.value = '';
    connectWebSocket(token);

    await refreshFriendsData();
  } catch (error) {
    authCardRef.value?.setError(getErrorMessage(error, 'Unable to reach the backend.'));
  }
};

const handleSendFriendRequest = async (receiverId: string) => {
  actionError.value = '';
  actionSuccess.value = '';

  try {
    await authFetch<FriendRequest>('/friends/requests', {
      method: 'POST',
      body: JSON.stringify({ receiverId }),
    });
    actionSuccess.value = `Friend request sent to ${receiverId}.`;
    await refreshFriendsData();
  } catch (error) {
    actionError.value = getErrorMessage(error, 'Failed to send friend request.');
  }
};

const handleAcceptRequest = async (requestId: string) => {
  actionError.value = '';
  actionSuccess.value = '';

  try {
    await authFetch<{ accepted: boolean }>(`/friends/requests/${encodeURIComponent(requestId)}/accept`, {
      method: 'POST',
    });
    actionSuccess.value = 'Friend request accepted.';
    await refreshFriendsData();
  } catch (error) {
    actionError.value = getErrorMessage(error, 'Failed to accept friend request.');
  }
};

const handleDeclineRequest = async (requestId: string) => {
  actionError.value = '';
  actionSuccess.value = '';

  try {
    await authFetch<null>(`/friends/requests/${encodeURIComponent(requestId)}/decline`, {
      method: 'DELETE',
    });
    actionSuccess.value = 'Friend request declined.';
    await refreshFriendsData();
  } catch (error) {
    actionError.value = getErrorMessage(error, 'Failed to decline friend request.');
  }
};

const handleRemoveFriend = async (friendId: string) => {
  actionError.value = '';
  actionSuccess.value = '';

  try {
    await authFetch<null>(`/friends/${encodeURIComponent(friendId)}`, {
      method: 'DELETE',
    });
    actionSuccess.value = `Removed ${friendId} from your friends list.`;
    await refreshFriendsData();
  } catch (error) {
    actionError.value = getErrorMessage(error, 'Failed to remove friend.');
  }
};

const toggleAuthMode = () => {
  authMode.value = authMode.value === 'login' ? 'register' : 'login';
};

const handleLogout = () => {
  disconnectWebSocket();
  localStorage.removeItem('gomessenger_token');
  localStorage.removeItem('gomessenger_username');
  currentUser.value = '';
  currentUserId.value = '';
  friends.value = [];
  pendingRequests.value = [];
  selectedContactId.value = null;
  actionError.value = '';
  actionSuccess.value = '';
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

    <div v-else class="relative mx-auto flex h-screen max-w-[1600px] bg-gradient-to-br from-indigo-50 via-purple-50 to-pink-50 shadow-2xl">
      <button
        type="button"
        class="absolute right-3 top-3 z-10 inline-flex items-center gap-1 rounded-full bg-white/90 px-3 py-2 text-xs font-semibold text-slate-700 shadow-md transition hover:bg-white"
        @click="handleLogout"
      >
        <LogOut :size="14" />
        Logout {{ currentUser }}
      </button>

      <ChatSidebar
        :contacts="contacts"
        :pendingRequests="sidebarPendingRequests"
        :selectedContactId="selectedContactId"
        :isLoading="isFriendsLoading"
        :currentUserId="currentUserId"
        :actionError="actionError"
        :actionSuccess="actionSuccess"
        @selectContact="handleSelectContact"
        @sendFriendRequest="handleSendFriendRequest"
        @acceptRequest="handleAcceptRequest"
        @declineRequest="handleDeclineRequest"
        @removeFriend="handleRemoveFriend"
      />

      <div v-if="selectedContact" class="flex flex-1 flex-col">
        <ChatHeader
          :name="selectedContact.name"
          :avatar="selectedContact.avatar"
          status="friend connected"
          :online="false"
        />
        <ChatMessages :messages="currentMessages" />
        <div class="border-t border-indigo-100 bg-amber-50 px-6 py-3 text-sm text-amber-900">
          Friends are loaded from the backend. Chat message delivery is not wired in this screen yet.
        </div>
        <MessageInput disabled placeholder="Messaging integration is pending" />
      </div>

      <div v-else class="flex flex-1 items-center justify-center">
        <div class="text-center">
          <div class="mx-auto mb-4 flex size-16 items-center justify-center rounded-full bg-indigo-100">
            <MessageCirclePlus :size="32" class="text-indigo-600" />
          </div>
          <p class="font-medium text-gray-600">No friend selected</p>
          <p class="mt-1 text-sm text-gray-400">Send or accept a friend request from the sidebar.</p>
        </div>
      </div>
    </div>
  </main>
</template>
