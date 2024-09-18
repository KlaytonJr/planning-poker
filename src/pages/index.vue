<template>
  <div>
    <h1>Votação Fibonacci</h1>
    
    <!-- Se o usuário não entrou com um nome, pedir para entrar -->
    <div v-if="!username">
      <input v-model="tempUsername" placeholder="Digite seu nome" />
      <button @click="joinRoom">Entrar</button>
    </div>
    
    <!-- Interface de votação para quem já entrou -->
    <div v-else>
      <h2>Olá, {{ username }}</h2>
      
      <!-- Se ainda não votou, mostre as opções de voto -->
      <div v-if="!voted">
        <h3>Escolha seu voto:</h3>
        <button v-for="value in fibonacci" :key="value" @click="sendVote(value)">
          {{ value }}
        </button>
      </div>

      <!-- Mostre o voto se já votou -->
      <div v-if="voted">
        <h3>Você votou: {{ userVote }}</h3>
      </div>

      <!-- Botão para o moderador revelar os votos -->
      <div v-if="isModerator">
        <button @click="revealVotes">Revelar Votos</button>
      </div>

      <!-- Lista de usuários com seus votos -->
      <h3>Votos em tempo real:</h3>
      <ul>
        <li v-for="user in users" :key="user.id">
          {{ user.name }}: 
          <span v-if="votesVisible">{{ user.vote }}</span>
          <span v-else>{{ user.vote ? 'Votou' : 'Aguardando voto...' }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import io from 'socket.io-client';

export default {
  data() {
    return {
      socket: null,
      tempUsername: '', // Nome temporário antes de entrar
      username: null, // Nome de usuário após entrar
      fibonacci: [1, 2, 3, 5, 8, 13, 21],
      users: [], // Lista de usuários e votos
      votesVisible: false, // Controla a visibilidade dos votos
      voted: false, // Se o usuário já votou ou não
      userVote: null, // Armazena o voto do usuário
      isModerator: false // Se o usuário é moderador
    };
  },
  mounted() {
    // Conectando ao servidor WebSocket
    this.socket = io('https://backend-wild-dust-389.fly.dev', {
      "transports": ['websocket', "polling"]
    });
    // this.socket = io('http://localhost:3010/');

    // Recebe a lista de usuários atualizada
    this.socket.on('userListUpdate', (users) => {
      this.users = users;
    });

    // Controla a revelação dos votos
    this.socket.on('revealVotes', (visible) => {
      this.votesVisible = visible;
    });
  },
  methods: {
    joinRoom() {
      if (this.tempUsername) {
        this.username = this.tempUsername;
        this.socket.emit('join', this.username);

        // Verifica se o usuário é o moderador
        this.isModerator = this.users.length === 0; // O primeiro usuário será o moderador
      }
    },
    sendVote(value) {
      // Envia o voto para o servidor
      this.socket.emit('sendVote', value);
      this.userVote = value;
      this.voted = true;
    },
    revealVotes() {
      // O moderador decide revelar os votos
      this.socket.emit('revealVotes');
    }
  }
};
</script>
