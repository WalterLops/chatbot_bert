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
        <InputField @user-message="handleUserMessage" />
      </div>
    </div>
  </div>
</template>

<script>
import io from 'socket.io-client';
import InputField from './components/InputField.vue';

export default {
  data() {
    return {
      socket: null,
      messageHistory: [], // Array para armazenar histórico de mensagens
      selectedMode: '0', // Modo selecionado no menu
      isConnected: false, // Status da conexão WebSocket
    };
  },
  created() {
    this.socket = io('http://localhost:5000');
    this.socket.on('connect', () => {
      this.isConnected = true;
    });
    this.socket.on('disconnect', () => {
      this.isConnected = false;
    });
    this.socket.on('msg_response', (responseBody) => {
      const messageId = Date.now() + 1; // Usar um identificador único para a mensagem
      const lastMessageIndex = this.messageHistory.findIndex(message => message.id === messageId); // Encontre o índice da última mensagem
      if (lastMessageIndex !== -1) {
        this.messageHistory[lastMessageIndex] = { id: messageId, text: responseBody.resposta, fromUser: false, loading: false };
      }
    });
  },
  beforeUnmount() {
    if (this.socket) {
      this.socket.close();
    }
  },
  methods: {
    async handleUserMessage(userInput) {
      const messageId = Date.now();
      this.messageHistory.push({ id: messageId, text: userInput, fromUser: true, loading: false });
      this.messageHistory.push({ id: messageId + 1, text: "Carregando...", fromUser: false, loading: true });

      let lastMessageIndex = this.messageHistory.findIndex(message => message.id === messageId + 1); // Declaração movida para fora do try

      try {
        // Enviar mensagem para o servidor WebSocket
        this.socket.emit('EnviarMsg', { mensagem: userInput, modo: this.selectedMode });

        // Aguardar a resposta do servidor WebSocket
        this.socket.on('msg_response', (responseBody) => {
          this.messageHistory[lastMessageIndex] = { id: messageId + 1, text: responseBody.resposta, fromUser: false, loading: false };
        });
      } catch (error) {
        console.error("Erro na chamada para o back-end:", error);
        this.messageHistory.splice(lastMessageIndex, 1); // Remove o indicador de carregamento
      }
    }
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
