<template>
  <div>
    <h1 class="text-center mt-4">Votação Fibonacci</h1>
    
    <section class="container">

      <!-- Se o usuário não entrou com um nome, pedir para entrar -->
      <div v-if="!username">
        <input v-model="tempUsername" placeholder="Digite seu nome" />
        <button @click="joinRoom">Entrar</button>
      </div>
      
      <!-- Interface de votação para quem já entrou -->
      <div v-else class="voting">
        <h2 class="mb-6">Olá, {{ username }}</h2>
        
        <!-- Se ainda não votou, mostre as opções de voto -->
        <div>
          <h3>Escolha seu voto:</h3>
          <button v-for="value in fibonacci" :key="value" @click="sendVote(value)" :class="['options-vote mt-4', { selected: userVote === value }]">
            {{ value }}
          </button>
        </div>
  
        <!-- Botão para o moderador revelar os votos -->
        <div v-if="isModerator" class="d-flex items-center justify-center w-100 my-4 ga-4">
          <button @click="revealVotes" class="reveal-btn">Revelar Votos</button>
          <button v-if="complete" @click="resetVotes" class="reset-btn">Reiniciar Votação</button>
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
    </section>
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
      complete: false, // Se o usuário já votou ou não
      userVote: null, // Armazena o voto do usuário
      isModerator: false // Se o usuário é moderador
    };
  },
  mounted() {
    // Conectando ao servidor WebSocket
    this.socket = io('wss://backend-wild-dust-389.fly.dev', {
      "transports": ['websocket', "polling"]
    });
    // this.socket = io('http://localhost:3010');

    // Recebe a lista de usuários atualizada
    this.socket.on('userListUpdate', (users) => {
      this.users = users;
    });

    // Controla a revelação dos votos
    this.socket.on('revealVotes', (visible) => {
      this.votesVisible = visible;
      this.complete = true;
    });

    this.socket.on('resetVotes', () => {
      this.userVote = null;
      this.voted = false;
      this.complete = false;
      this.votesVisible = false;
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
    },
    resetVotes() {
      // O moderador decide limpar os votos e reiniciar a votação
      this.socket.emit('resetVotes'); // Envia o evento para o servidor
    }
  }
};
</script>

<style>
.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;

  height: 90vh;
  width: 100%;
}

.voting {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

input {
  border: 1px solid white;
  border-radius: 5px;
  padding: 5px 10px;
  margin: 0 15px;
}

.options-vote {
  border: 1px solid white;
  border-radius: 50px;
  width: 50px;
  height: 50px;
  margin: 0 15px;
}

.selected {
  background-color: #fff;
  color: #000;
  border-color: #fff;
}

.reveal-btn {
  border: 1px solid white;
  border-radius: 5px;
  padding: 5px 10px;
}

.reset-btn {
  border: 1px solid white;
  background-color: #fff;
  color: #000;
  border-radius: 5px;
  padding: 5px 10px;
}
</style>