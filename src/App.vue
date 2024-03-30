<template>
  <div class="background-container" v-chat-scroll>
    <div class="chat-container" v-chat-scroll>
      <!-- Renderize as mensagens existentes -->
      <div v-for="message in messageHistory" :key="message.id" class="message">
        <div v-if="message.fromUser" class="user-message">
          {{ message.text }}
        </div>
        <div v-else class="server-message">
          {{ message.text }}
        </div>
      </div>
      <!-- Componente InputField para o campo de entrada -->
      <InputField @user-message="handleUserMessage" />
    </div>
  </div>
</template>

<script>
import InputField from './components/InputField.vue';

export default {
  data() {
    return {
      messageHistory: [], // Array para armazenar histórico de mensagens
    };
  },
  methods: {
    async handleUserMessage(userInput) {
      try {
        const responseFromBackend = await fetch("https://66db-35-245-150-41.ngrok-free.app/EnviarMsg", {
          method: "POST",
          body: JSON.stringify({ mensagem: userInput }),
          headers: {
            "Content-Type": "application/json",
          },
        });

        if (responseFromBackend.ok) {
          const responseBody = await responseFromBackend.json();
          this.messageHistory.push({ id: Date.now(), text: userInput, fromUser: true });
          this.messageHistory.push({ id: Date.now() + 1, text: responseBody.resposta, fromUser: false });
        } else {
          console.error("Erro na resposta do servidor:", responseFromBackend.status);
        }
      } catch (error) {
        console.error("Erro na chamada para o back-end:", error);
      }
    },
  },
  components: {
    InputField,
  },
};
</script>

<style scoped>

.chat-container {
  max-width: 66.67%; /* 2/3 do espaço */
  margin: 0 auto;
  padding: 20px;
  background-color: #20202070; 
  border-radius: 10px;
  position: relative;
  background-size: cover;
  height: 100vh;
  overflow-y: auto;
}

.message {
  margin-bottom: 10px;
}

.user-message,
.server-message {
  display: inline-block; /* Ajusta ao tamanho do texto */
  padding: 10px;
  border-radius: 10px;
  color: white;
  margin-top: 10px;
}

.user-message {
  background-color: #128C7E;
  text-align: right;
  margin-left: auto;
  float: right;
  /* padding-bottom: 4px;
  padding-top: 4px; */
}

.server-message {
  background-color: #333030e3;
  text-align: left;
}

.background-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: url("https://img.freepik.com/vetores-gratis/conceito-de-fundo-ondulado_23-2148497712.jpg?size=626&ext=jpg");
  background-size: cover;
  overflow-y: auto;
}
</style>
