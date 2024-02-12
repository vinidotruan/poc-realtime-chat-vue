<script setup lang="ts">
  import {ref} from "vue";
  import Echo from 'laravel-echo';
  import Pusher from 'pusher-js';
  window.Pusher = Pusher;
  let laravelEcho: Echo;
  let channelListener: any;

  const message = ref('');
  const makeLogin = () => {
    fetch("http://localhost:8000/api/v1/login", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        email: "test@teste.com",
        password: "password",
      })
    }).then((response) => {
      return response.json();
    }).then((data) => {
      localStorage.setItem('token', data.token);
      localStorage.setItem('user_id', data.profile_id);
      laravelEcho = new Echo({
        broadcaster: 'pusher',
        key:'app-key',
        wsHost: 'localhost',
        wsPort: 6001,
        wssPort: 6001,
        forceTLS: false,
        encrypted: false,
        enabledTransports: ['ws', 'wss'],
        cluster: 'mt1',
        authEndpoint: 'http://localhost:8000/broadcasting/auth',
        auth: {
          headers: {
            Authorization: 'Bearer ' + localStorage.getItem('token'),
            Accept: "application/json",
          },
        },
      });

    }).catch((error) => {
      console.log(error);
    });
  };

  const sendMessage = () => {
    const chatId = "9b51f7dc-a3b8-434a-8e04-1a27d0fe7b27"
    const body = {
      message: message.value,
      profile_id: localStorage.getItem('user_id'),
      reply_to: localStorage.getItem('user_id'),
      chat_id: chatId,
    };
    fetch("http://localhost:8000/api/v1/chat", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": "Bearer " + localStorage.getItem('token'),
        "X-Socket-ID": laravelEcho.socketId(),
      },
      body: JSON.stringify(body),
    }).then(response => {
      return response.json();
    }).then(data => {
      console.log(data)
      localStorage.setItem("chat_id", data.chat);
      console.log("chat."+data.chat)
      if(!channelListener) {
        channelListener = laravelEcho.private('chat.'+chatId).listen('.message-sent', (e) => {
          console.log(e);
        });
      }
    });
  };

  makeLogin();


</script>

<template>
  <header>
    <div class="wrapper">
    </div>
  </header>

  <main>
    <div class="container-fluid">
      <div class="row">
        <div class="col-12">
          <form>
            <div class="mb-3">
              <label for="message" class="form-label">Digite seu texto para mensagem</label>
              <input type="text" class="form-control" id="message"  v-model="message">
            </div>
            <button type="button" class="btn btn-primary" @click="sendMessage">Enviar Mensagem</button>
          </form>
        </div>
      </div>
      <div class="row">
        <div class="col-12">
          <span> {{ message }}</span>
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>

</style>
