<template>
  <div class="container">
    <v-sheet>
      <div>
        <VForm
          @submit.prevent="sendMessage"
          v-model="isFormValid"
          class="chat-input"
        >
          <VTextField
            label="Username"
            color="primary"
            variant="outlined"
            :readonly="!firstMessage"
            class="input mb-10"
            :rules="rules"
            v-model="message.username"
          ></VTextField>

          <VTextarea
            label="Message"
            variant="outlined"
            :rules="rules"
            v-model="message.value"
            class="input textarea"
          ></VTextarea>

          <VBtn type="submit" :disabled="!isFormValid" color="primary"
            >SEND</VBtn
          >
        </VForm>
      </div>
    </v-sheet>
    <div class="messages">
      <v-list lines="one">
        <v-list-item
          class="mb-2"
          v-for="m in messages"
          :title="m.username"
          :subtitle="m.value"
        ></v-list-item>
      </v-list>
    </div>
  </div>
</template>

<script lang="ts">
import * as signalR from "@microsoft/signalr";
type Message = {
  username: string;
  value: string;
};

export default {
  mounted() {
    this.connection.start().catch((err) => console.log(err));

    this.connection.on("RecieveMessage", (message: Message) => {
      this.messages.push(message);
    });
  },
  data() {
    return {
      firstMessage: true,
      message: {} as Message,
      messages: [] as Message[],
      connection: new signalR.HubConnectionBuilder()
        .withUrl("https://chathub20240302193545.azurewebsites.net/messages")
        .build(),
      rules: [
        (value: string) => {
          if (value) return true;

          return "Field is required";
        },
      ],
      isFormValid: false,
    };
  },
  methods: {
    sendMessage() {
      if (this.firstMessage) !this.firstMessage;
      this.connection
        .invoke("SendMessage", this.message)
        .catch((err) => console.log(err));
    },
  },
};
</script>

<style scoped>
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;

  gap: 128px;
}

.input {
  width: 350px;
  max-height: 50px;
}

.chat-input {
  display: flex;
  flex-direction: column;
}
.textarea {
  margin-bottom: 130px;
}

.messages {
  overflow-y: scroll;
  height: 300px;
  width: 500px;
  overflow: auto;
}
</style>
