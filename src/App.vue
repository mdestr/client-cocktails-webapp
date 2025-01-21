<template>
  <div>
    <h1>Liste des Clients</h1>
    <ul>
      <li v-for="client in clients" :key="client.id">
        <!-- Nom du client avec un lien pour afficher/masquer les détails -->
        <a href="#" @click.prevent="toggleClientDetails(client)">
          {{ client.name }}
        </a>

        <!-- Section pour afficher les détails du client -->
        <div v-if="client.showDetails" class="client-details">
          <p><strong>Nom:</strong> {{ client.details?.name || "Chargement..." }}</p>
          <p><strong>Nom traduit :</strong> {{ client.details?.translatedName || "Chargement..." }}</p>
          <p><strong>Domain:</strong> {{ client.details?.domain || "Chargement..." }}</p>
          <p><strong>Twitter:</strong> {{ client.details?.twitterUsername || "Chargement..." }}</p>
          <p><strong>Followers:</strong> {{ client.details?.followersCount || "Chargement..." }}</p>
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      clients: [], // Liste des clients avec showDetails et details
    };
  },
  mounted() {
    // Requête pour récupérer la liste des clients
    axios
      .get("http://localhost:8080/clients")
      .then((response) => {
        // Initialise les clients avec une propriété `showDetails` et un espace pour les détails
        this.clients = response.data._embedded.clientSummaryDTOList.map((client) => ({
          ...client,
          showDetails: false, // Par défaut, les détails ne sont pas affichés
          details: null, // Espace pour stocker les détails
        }));
      })
      .catch((error) => {
        console.error("Erreur lors de la récupération des clients:", error);
      });
  },
  methods: {
    // Méthode pour afficher/masquer les détails d'un client
    toggleClientDetails(client) {
      // Si les détails sont déjà visibles, on les masque
      if (client.showDetails) {
        client.showDetails = false;
        return;
      }

      // Sinon, on les affiche et récupère les données si elles ne sont pas déjà chargées
      client.showDetails = true;

      if (!client.details) {
        axios
          .get(`http://localhost:8080/clients/${client.id}`)
          .then((response) => {
            client.details = response.data; // Ajoute les détails au client
          })
          .catch((error) => {
            console.error("Erreur lors de la récupération des détails du client:", error);
          });
      }
    },
  },
};
</script>

<style scoped>
/* Style pour différencier les détails */
.client-details {
  margin-left: 20px;
  padding: 10px;
  border-left: 2px solid #ccc;
  color: #333;
  background-color: #f9f9f9;
  border-radius: 5px;
}
</style>
