<template>
  <div class="input-field">
    <input 
      v-model="userInput" 
      @input="checkWordLimit" 
      @keyup.enter="sendMessage" 
      placeholder="Digite sua mensagem..." 
    />
    <button @click="sendMessage">Enviar</button>
    <div class="status-container">
      <div :class="['ball', isConnected ? 'green' : 'red']"></div>
      <div class="word-count">
        <span>{{ wordCount }}/{{ wordLimit }} palavras</span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: ['isConnected'],
  data() {
    return {
      userInput: "",
      wordLimit: 22, // Define o limite de palavras aqui
    };
  },
  computed: {
    wordCount() {
      return this.userInput.trim().split(/\s+/).filter(word => word).length;
    }
  },
  methods: {
    sendMessage() {
      if (this.wordCount <= this.wordLimit) {
        this.$emit("user-message", this.userInput);
        this.userInput = "";
      }
    },
    checkWordLimit() {
      const words = this.userInput.trim().split(/\s+/).filter(word => word);
      if (words.length > this.wordLimit) {
        this.userInput = words.slice(0, this.wordLimit).join(" ");
      }
    },
  },
};
</script>

<style scoped>
.input-field {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  padding: 16px;
}

input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  margin-bottom: 8px;
}

button {
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 4px;
  padding: 8px 12px;
  cursor: pointer;
  margin-bottom: 8px;
}

.status-container {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 8px;
}

.ball {
  width: 20px;
  height: 20px;
  border-radius: 100%;
  margin-right: 8px;
}

.green {
  background-color: green;
}

.red {
  background-color: red;
}

.word-count {
  color: white;
  align-items: center;
}

@media (min-width: 600px) {
  .input-field {
    flex-direction: row;
    align-items: center;
  }

  input {
    margin-bottom: 0;
    margin-right: 8px;
  }

  button {
    margin-bottom: 0;
    margin-right: 8px;
  }

  .status-container {
    margin-bottom: 0;
    margin-right: 8px;
    justify-content: flex-start;
    align-items: center;
  }
}
</style>
