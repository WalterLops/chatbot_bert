/**/
<template>
  <div class="background-container">
    <div class="chat-container">
      <!-- Menu para seleção do modo -->
      <div class="menu-container">
        <select v-model="selectedMode">
          <option value="0">Modo BERT base</option>
          <option value="1">Modo BERT large</option>
        </select>
      </div>
      <!-- Renderize as mensagens existentes -->
      <div class="messages-container">
        <div v-for="message in messageHistory" :key="message.id" class="message">
          <div v-if="message.loading" class="server-message">
            Carregando...
          </div>
          <div v-else-if="message.fromUser" class="user-message">
            {{ message.text }}
          </div>
          <div v-else class="server-message">
            {{ message.text }}
          </div>
        </div>
      </div>
      <!-- Componente InputField para o campo de entrada -->
      <div class="input-field-container">
        <InputField :isConnected="isConnected" @user-message="handleUserMessage" />
      </div>
    </div>
  </div>
</template>

<script>
import InputField from './components/InputField.vue';
import { io } from 'socket.io-client';

export default {
  data() {
    return {
      socket: null,
      messageHistory: [],
      selectedMode: '0',
      isConnected: false,
    };
  },
  created() {
    const ngrokUrl = 'https://b3ed-34-105-87-204.ngrok-free.app';
    this.socket = io(ngrokUrl, { transports: ['websocket'] });

    this.socket.on('connected', () => {
      this.isConnected = true;
      console.log('Connected to WebSocket server.');
    });

    this.socket.on('disconnected', () => {
      this.isConnected = false;
      console.log('Disconnected from WebSocket server.');
    });

    this.socket.on('connection_response', (data) => {
      console.log('Connection response:', data);
    });

    this.socket.on('connection_response', (data) => {
      if (data.status === 'connected') {
        this.isConnected = true;
        console.log('Connected to WebSocket server.');
      } else {
        this.isConnected = false;
        console.log('Disconnected from WebSocket server.');
      }
    });


    this.socket.on('msg_response', (responseBody) => {
      const messageId = Date.now() + 1;
      console.log('Received message response:', responseBody);
      const lastMessageIndex = this.messageHistory.findIndex(message => message.id === messageId);
      if (lastMessageIndex !== -1) {
        this.messageHistory[lastMessageIndex] = { id: messageId, text: responseBody.resposta, fromUser: false, loading: false };
      } else {
        this.messageHistory.push({ id: messageId, text: responseBody.resposta, fromUser: false, loading: false });
      }
    });
  },
  beforeUnmount() {
    if (this.socket) {
      this.socket.close();
    }
  },
  methods: {
    handleUserMessage(userInput) {
      const messageId = Date.now();
      this.messageHistory.push({ id: messageId, text: userInput, fromUser: true, loading: true });

      try {
        console.log('Sending message to WebSocket:', userInput);
        this.socket.emit('EnviarMsg', { id: messageId, mensagem: userInput, modo: this.selectedMode });
      } catch (error) {
        console.error("Erro na chamada para o back-end:", error);
        this.messageHistory = this.messageHistory.filter(message => message.id !== messageId);
      }
    },
  },
  components: {
    InputField,
  },
};
</script>

<style scoped>
.menu-container {
  margin: 20px;
  text-align: center;
}

.chat-container {
  max-width: 66.67%;
  margin: 0 auto;
  padding: 20px;
  background-color: #20202070;
  border-radius: 10px;
  position: relative;
  background-size: cover;
  height: 100vh;
  display: flex;
  flex-direction: column;
}

.messages-container {
  overflow-y: auto;
  flex-grow: 1;
  margin-bottom: 10px;
  /* Espaço extra para o último item */
}

.message {
  margin-bottom: 10px;
}

.user-message,
.server-message {
  display: block;
  /* Alterado de inline-block para block */
  padding: 10px;
  border-radius: 10px;
  color: white;
  margin-top: 5px;
  margin-bottom: 5px;
  width: auto;
  /* Garante que ocupe a largura disponível */
  clear: both;
  /* Resolve problemas quando se misturam float e block */
}

.user-message {
  background-color: #128C7E;
  text-align: right;
  margin-left: auto;
  float: right;
}

.server-message {
  background-color: #333030e3;
  text-align: left;
}

.input-field-container {
  margin-top: auto;
  /* Mantém na parte inferior */
  padding-bottom: 40px;
  /* Espaço extra para o input */
}

.background-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: url("https://img.freepik.com/vetores-gratis/conceito-de-fundo-ondulado_23-2148497712.jpg?size=626&ext=jpg");
  background-size: cover;
}

.server-message,
.loading-message {
  /* Estilo para mensagens de carregamento */
  background-color: #333030e3;
  text-align: left;
  float: left;
  /*margin-left: 20px;  Garante um pouco de espaço no lado esquerdo */
}
</style>
