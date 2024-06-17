/**/
<template>
  <div class="background-container">
    <div class="chat-container">
      <div class="header">
        <h1>ChatBERT</h1>
      </div>
      <!-- Renderize as mensagens existentes -->
      <div class="messages-container">
        <div v-for="message in messageHistory" :key="message.id"
          :class="['message', { 'user-message': message.fromUser, 'server-message': !message.fromUser }]">
          <div v-if="message.loading">
            Carregando...
          </div>
          <div v-else>
            <div v-if="message.msg">
              <div><strong>Mensagem:</strong> {{ message.msg.mensagem }}</div>
              <div class="tempo"><strong>Tempo:</strong> {{ message.msg.tempoMsg }}</div>
            </div>
            <div v-else>
              <div>{{ message.text }}</div>
            </div>
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
import { io } from "socket.io-client";

export default {
  components: {
    InputField,
  },
  data() {
    return {
      socket: null,
      messageHistory: [],
      selectedMode: '0',
      isConnected: false,
      ngrokUrl: '',
      loadingId: 0
    };
  },
  created() {
    this.loadConfig().then(() => {
      this.connectWebSocket();
    });
  },
  methods: {
    async loadConfig() {
      const response = await fetch('/config.json');
      const config = await response.json();
      this.apiKey = config.apiKey;
      this.clientId = config.clientId;
      this.folderId = config.folderId;
      this.ngrokUrl = config.ngrokUrl;
    },
    connectWebSocket() {
      console.log(`Connecting to WebSocket at: ${this.ngrokUrl}`);
      this.socket = io(this.ngrokUrl, {
        transports: ['websocket']
      });

      this.socket.on('connect', () => {
        this.isConnected = true;
        console.log('Connected to WebSocket server.');
      });

      this.socket.on('disconnect', () => {
        this.isConnected = false;
        console.log('Disconnected from WebSocket server.');
      });

      this.socket.on('connection_response', (data) => {
        if (data.status === 'connected') {
          this.isConnected = true;
        } else {
          this.isConnected = false;
        }
        console.log('Connection response:', data);
      });

      this.socket.on('msg_response', (responseBody) => {

        const { mensagem, tempoMsg } = responseBody;
        const msg = { mensagem, tempoMsg };

        const messageIndex = this.messageHistory.findIndex(message => message.id === this.loadingId);

        if (messageIndex !== -1) {
          this.messageHistory[messageIndex] = { id: responseBody.id, text: '', fromUser: false, loading: false, msg };
        } else {
          this.messageHistory.push({ id: responseBody.id, text: '', fromUser: false, loading: false, msg });
        }

      });
    },
    handleUserMessage(userInput) {
      const messageId = Date.now();
      this.loadingId = `loading-${messageId}`;
      this.messageHistory.push({ id: messageId, text: userInput, fromUser: true, loading: false });
      this.messageHistory.push({ id: this.loadingId, text: 'Carregando...', fromUser: false, loading: true });

      try {
        console.log('Sending message to WebSocket:', userInput);
        this.socket.emit('EnviarMsg', { id: messageId, mensagem: userInput, modo: this.selectedMode });
      } catch (error) {
        console.error("Erro na chamada para o back-end:", error);
        this.messageHistory = this.messageHistory.filter(message => message.id !== this.loadingId); // Remove a mensagem de carregamento
      }
    },
  },
}; 
</script>


<style scoped>

.header {
  text-align: center;
  padding: 10px;
  background-color: #f5f5f5;
  border-bottom: 1px solid #ddd;
}

h1 {
  margin: 0;
  font-size: 24px;
  color: #333;
}

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
  /*-image: url("https://img.freepik.com/vetores-gratis/conceito-de-fundo-ondulado_23-2148497712.jpg?size=626&ext=jpg");*/
  background-color: rgba(12, 11, 11, 0.815);
  background-size: cover;
}

.server-message,
.loading-message {
  /* Estilo para mensagens de carregamento */
  background-color: #484545e3;
  text-align: left;
  float: left;
  /*margin-left: 20px;  Garante um pouco de espaço no lado esquerdo */
}

.message-history {
  margin-top: 20px;
}

.user-message {
  text-align: right;
}

.tempo{
  font-size: 14px;
}

.response-message {
  text-align: left;
}
</style>
