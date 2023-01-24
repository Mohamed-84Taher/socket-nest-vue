<script setup>
import { io } from 'socket.io-client';
import { onBeforeMount, ref } from 'vue';

const socket = io('http://localhost:3001');

const messages = ref([]);
const messageText = ref('');
const name = ref('');
const joined = ref(false);
const typingDispaly = ref('');

onBeforeMount(() => {
  socket.emit('findAllMessages', {}, (response) => {
    messages.value = response;
  });

  socket.on('message', (message) => {
    messages.value.push(message);
  });

  socket.on('typing', ({ name, isTyping }) => {
    if (isTyping) {
      typingDispaly.value = `${name} is typing`;
    } else {
      typingDispaly.value = '';
    }
  });
});

const join = () => {
  socket.emit('join', { name: name.value }, () => {
    joined.value = true;
  });
};

const sendMessage = () => {
  socket.emit('createMessage', { text: messageText.value }, (response) => {
    messageText.value = '';
  });
};

let timeout;
const emitTyping = () => {
  socket.emit('typing', { isTyping: true });
  timeout = setTimeout(() => {
    socket.emit('typing', { isTyping: false });
  }, 2000);
};
</script>

<template>
  <div class="chat">
    <div v-if="!joined">
      <form @submit.prevent="join">
        <label> what's your name ? </label><br />
        <input v-model="name" /><br />
        <button type="submit">Send</button>
      </form>
    </div>
    <div class="chat-container" v-else>
      <div class="messages-container">
        <div v-for="message in messages">
          [{{ message.name }}]:[{{ message.text }}]
        </div>
      </div>

      <div v-if="typingDispaly">
        {{ typingDispaly }}
      </div>

      <hr />
      <div class="message-input">
        <form @submit.prevent="sendMessage">
          <label> Message </label><br />
          <input v-model="messageText" @input="emitTyping" /><br />
          <button type="submit">Send</button>
        </form>
      </div>
    </div>
  </div>
</template>

<style scoped>
@import './assets/base.css'
</style>
