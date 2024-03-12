<script setup lang="ts">
  import {ref} from "vue";
  import Echo from 'laravel-echo';
  import Pusher from 'pusher-js';
  window.Pusher = Pusher;
  let laravelEcho: Echo;
  let channelListener: any;
  const wsHost = 'ec2-3-84-24-193.compute-1.amazonaws.com';
  const wsHostLocal = 'localhost';
  const message = ref('');
  const receiverId = ref('');
  const receivedMessages = ref([]);

  const makeLogin = () => {
    fetch("http://localhost:8000/api/v1/login", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        email: "teste@teste.com",
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
        wsHost: wsHostLocal,
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

  const makeChat = () => {
    fetch("http://localhost:8000/api/v1/chat/create", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": "Bearer " + localStorage.getItem('token'),
      }
    }).then(response => {
      return response.json();
    }).then(data => {
      console.log(data)
      localStorage.setItem("chat_id", data.chat);
      console.log("chat."+data.chat)
        channelListener = laravelEcho.private('chat.'+data.chat).listen('.message-sent', (e) => {
          console.log(e);
          receivedMessages.value.push(e);
        });

    })
  }

  const sendMessage = () => {
    const chatId = localStorage.getItem('chat_id')
    const body = {
      message: message.value,
      profile_id: localStorage.getItem('user_id'),
      reply_to: receiverId.value,
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
      <div class="row d-flex justify-content-center align-items-center">
        <div class="col-6">
          <form>
            <div class="mb-3">
              <label for="message" class="form-label">Digite o ID para quem deseja enviar a mensagem</label>
              <input type="text" class="form-control" id="message" v-model="receiverId">
            </div>
            <div class="mb-3">
              <label for="message" class="form-label">Digite seu texto para mensagem</label>
              <input type="text" class="form-control" id="message" v-model="message">
            </div>
            <button type="button" class="btn btn-primary" @click="makeChat">Criar Chat</button>
            <button type="button" class="btn btn-primary mx-4" @click="sendMessage">Enviar Mensagem</button>
          </form>
        </div>
      </div>
      <div class="row d-flex justify-content-center align-items-center">
        <div class="col-6 d-flex justify-content-start align-items-center flex-column text-white">
          <div class="align-self-start" >
          <span class="badge rounded-pill text-bg-primary"> enviada:</span>
           {{ message }}
          </div>
          <div class="align-self-start" >
            <span class="badge rounded-pill text-bg-warning">received: </span>
            {{ receivedMessages }}
          </div>

        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>

</style>
