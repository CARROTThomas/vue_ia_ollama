<script setup lang="ts">
import { ref } from 'vue';
import axios from 'axios';

// Définition de l'interface Message
interface Message {
  text: string;
  from: 'user' | 'ia';
}

const apiUrl = 'http://127.0.0.1:11434/api/generate';

const prompt = ref('');
const messages = ref<Message[]>([]);

const envoyerPrompt = async () => {
  try {
    // Envoyer le message de l'utilisateur
    messages.value.push({ text: prompt.value, from: 'user' });

    // Obtenir la réponse de l'IA
    const response = await axios.post(apiUrl, {
      model: "mistral",
      stream: false,
      temperature: 0.7,
      max_tokens: -1,
      prompt: prompt.value
    });

    // Ajouter la réponse de l'IA
    messages.value.push({ text: response.data.response, from: 'ia' });

    // Vider l'input du prompt après l'envoi
    prompt.value = '';
  } catch (error) {
    console.error("Erreur lors de l'envoi du prompt à Ollama :", error);
  }
};
</script>


<template>
  <div class="container container-content-chat">
    <h1 class="text-light">Ollama Chat</h1>
    <div class="conversation">
      <div v-for="(message, index) in messages" :key="index" class="message" :class="message.from === 'user' ? 'user-message' : 'ia-message'">
        {{ message.text }}
      </div>
    </div>
    <form @submit.prevent="envoyerPrompt" class="mt-3">
      <div class="mb-3">
        <label for="prompt" class="form-label">Entrez votre prompt :</label>
        <input type="text" class="form-control" id="prompt" v-model="prompt">
      </div>
      <button type="submit" class="btn btn-primary">Envoyer</button>
    </form>
  </div>
</template>


<style scoped>
/* Styles pour la conversation de messages */
.conversation {
  height: 70vh;
  padding: 0 1.3em;
  min-width: 85%;
  max-width: 90%;
  margin: 20px auto;

  gap: 1em;
  display: flex;
  flex-direction: column;


  overflow: scroll;
}

.message {
  width: auto;
  max-width: 60%;
  min-width: auto;
  padding: 10px;
  border-radius: 10px;
  margin-bottom: 10px;
  word-wrap: break-word;
}

.user-message {
  text-align: right;
  background-color: #DCF8C6;
  align-self: flex-end; /* aligné à droite */
}

.ia-message {
  text-align: left;
  background-color: #E8E8E8;
  align-self: flex-start; /* aligné à gauche */
}
</style>
