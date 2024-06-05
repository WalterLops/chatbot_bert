<template>
  <div class="input-field">
    <input v-model="userInput" @keyup.enter="sendMessage" placeholder="Digite sua mensagem..." />
    <button @click="sendMessage">Enviar</button>
    <div :class="['ball', isConnected ? 'green' : 'red']"></div>
  </div>
</template>

<script>
export default {
  props: ['isConnected'],
  data() {
    return {
      userInput: "",
    };
  },
  methods: {
    sendMessage() {
      this.$emit("user-message", this.userInput);
      this.userInput = "";
    },
  },
};
</script>

<style scoped>
.input-field {
  display: flex;
  align-items: center;
}

input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  margin-left: 8px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 4px;
  padding: 8px 12px;
  cursor: pointer;
}

.ball {
  width: 20px;
  height: 20px;
  border-radius: 100%;
  margin: 15px;
}

.green {
  background-color: green;
}

.red {
  background-color: red;
}
</style>
