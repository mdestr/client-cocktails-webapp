<style scoped src="@/assets/css/styles.css"></style>
<template>
  <div>
    <h1>Liste des Clients</h1>
    <ul>
      <li v-for="client in clients" :key="client.id">
        <a href="#" @click="getClientDetails(client.id)">{{ client.name }}</a>
      </li>
    </ul>

    <!-- Section pour afficher les détails du client -->
    <div v-if="clientDetails">
      <h2>Détails du Client</h2>
      <p><strong>Nom:</strong> {{ clientDetails.name }}</p>
      <p><strong>Domain:</strong> {{ clientDetails.domain }}</p>
      <p><strong>Twitter:</strong> {{ clientDetails.twitterUsername }}</p>
      <p><strong>Followers:</strong> {{ clientDetails.followersCount }}</p>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      clients: [],           // Liste des clients
      clientDetails: null    // Détails du client
    };
  },
  mounted() {
    // Requête pour récupérer la liste des clients
    axios.get('http://localhost:8080/clients')
      .then(response => {
        this.clients = response.data._embedded.clientSummaryDTOList;
      })
      .catch(error => {
        console.error("Erreur lors de la récupération des clients:", error);
      });
  },
  methods: {
    // Méthode pour récupérer les détails d'un client
    getClientDetails(clientId) {
      axios.get(`http://localhost:8080/clients/${clientId}`)
        .then(response => {
          this.clientDetails = response.data;  // Assigne les détails du client à `clientDetails`
        })
        .catch(error => {
          console.error("Erreur lors de la récupération des détails du client:", error);
        });
    }
  }
}
</script>
