<template id="app">
  <v-container class="container--fluid">
    <v-row>
      <v-col cols="12" md="5" sm="12">
        <h1>AI ChatBot</h1>
        <p class="caption">
          This chatbot project from OpenAI is designed to demonstrate the capabilities of the GPT-3 natural language processing model, which has been trained on a massive corpus of human-written text. The chatbot allows users to have a conversation with an AI-powered agent that can generate human-like responses to a wide range of prompts and questions. The user can input their own text prompts and the chatbot will generate a response based on its training data. The project is built using Vue.js for the front-end, and utilizes the OpenAI API to interact with the GPT-3 model. Overall, the project showcases the remarkable advancements that have been made in natural language processing, and highlights the potential for AI-powered chatbots to enhance communication and improve the user experience in a wide range of applications.
        </p>
      </v-col>
      <v-col cols="12" md="7" sm="12">
        <v-textarea v-model="question" outlined rows="2" placeholder="Type your question here" @keyup.enter="generateText">
         <template v-slot:append>
    <v-btn small icon @click="sendQuestion">
      <v-icon small>mdi-send</v-icon>
    </v-btn>
  </template>
        </v-textarea>
        <v-textarea v-model="response" outlined readonly rows="10"></v-textarea>
         <v-btn @click="clearConversation" class="cbtn text-capitalize">Clear Conversation</v-btn>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import { Configuration, OpenAIApi } from "openai";

export default {
  data() {
    return {
      question: "",
      response: 'AI: Hello! How can I assist you today?\n\n',
    }
  },
  head: {
    title: 'AI ChatBot'
  },
  created() {
    if (!process.env.OPENAI_API_KEY) {
      console.error("OpenAI API key not found.");
      return;
    }
    const configuration = new Configuration({
      apiKey: process.env.OPENAI_API_KEY,
    });
    this.openai = new OpenAIApi(configuration);
    console.log("Connected to OpenAI API.");
  },
  methods: {
    async generateText() {
      const response = await this.openai.createCompletion({
        model: "text-davinci-003",
        prompt: "You: " + this.question + "\nAI:",
        temperature: 0,
        max_tokens: 1000,
        stop: ["You:"],
      });
      const message = "AI: " + response.data.choices[0].text + "\n\n";
      this.response += "You: " + this.question + "\n" + message;
      this.question = "";
    },
    clearConversation() {
      this.response = 'AI: Hello! How can I assist you today?\n\n';
      this.question = "";
    },
    sendQuestion() {
  if (this.question.trim()) {
    this.generateText();
  }
}
  },
};

</script>
<style scoped>
.v-main *{
  font-family: 'Quicksand', sans-serif !important;
}
textarea.full-width {
  width: 100%;
  height: 200px;
  border: 2px solid white;
}

.cbtn {
  background: linear-gradient(90deg, #00C9FF 0%, #92FE9D 100%);
}

</style>
