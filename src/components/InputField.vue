<template>
    <div class="input-field">
        <input v-model="userInput" @keyup.enter="sendMessage" placeholder="Digite sua mensagem..." />
        <button @click="sendMessage">Enviar</button>
    </div>
</template>

<script>
export default {
    data() {
        return {
            userInput: "", // Armazena a mensagem digitada pelo usuário
        };
    },
    methods: {
        sendMessage() {
            // Emitir evento para o componente pai (ChatBox.vue) com a mensagem digitada
            this.$emit("user-message", this.userInput);
            // Limpar o campo de entrada
            this.userInput = "";
        },
    },
};
</script>

<style scoped>
.input-field {
    display: flex;
    align-items: center;

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
}
</style>