<template>
  <div>
    <h1>Liste des Clients</h1>
    <ul>
      <li v-for="client in clients" :key="client.id">
        <a href="#" @click.prevent="toggleDetails(client.id)">
          {{ client.name }}
        </a>
        <!-- Ajouter une icône cocktail si le client a des cocktails -->
        <img
          v-if="client.hasCocktails"
          src="./assets/cocktail_882673.png"
          alt="Cocktail Icon"
          style="width: 24px; height: 24px; margin-left: 5px;"
        />

        <!-- Affichage des détails du client -->
        <div v-if="selectedClientId === client.id && clientDetails">
          <h3>Détails du Client</h3>
          <p><strong>Nom:</strong> {{ clientDetails.name }}</p>
          <p><strong>Id:</strong> {{ clientDetails.id }}</p>
          <p><strong>Nom traduit:</strong> {{ clientDetails.translatedName }}</p>
          <p><strong>Domain:</strong> {{ clientDetails.domain }}</p>
          <p><strong>Twitter:</strong> {{ clientDetails.twitterUsername }}</p>
          <p><strong>Followers:</strong> {{ clientDetails.followersCount }}</p>

          <!-- Liste des cocktails -->
          <h4>Cocktails</h4>
          <ul>
            <li v-for="cocktail in cocktails" :key="cocktail.id">
              <a href="#" @click.prevent="getCocktailDetails(cocktail.id)">
                {{ cocktail.name }}
              </a>
            </li>
          </ul>

          <!-- Détails du cocktail -->
          <div v-if="selectedCocktailId === cocktailDetails?.id">
            <h4>Détails du Cocktail</h4>
            <p><strong>Nom:</strong> {{ cocktailDetails.name }}</p>
            <p><strong>Catégorie:</strong> {{ cocktailDetails.category }}</p>
            <p><strong>Alcoolisé:</strong> {{ cocktailDetails.alcoholic }}</p>
            <p><strong>Verre:</strong> {{ cocktailDetails.glassType }}</p>
            <p><strong>Instructions:</strong> {{ cocktailDetails.instructions }}</p>
            <p><strong>Ingrédients:</strong> {{ cocktailDetails.ingredients }}</p>
          </div>
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
      clients: [],
      clientDetails: null,
      cocktails: [],
      cocktailDetails: null,
      selectedClientId: null,
      selectedCocktailId: null,
    };
  },
  mounted() {
    // Charger la liste des clients avec vérification des cocktails
    axios
      .get("http://localhost:8080/clients")
      .then((response) => {
        const clients = response.data._embedded.clientSummaryDTOList;

        // Vérifier si chaque client a des cocktails
        const clientPromises = clients.map((client) => {
          return axios
            .get(client._links.self.href)
            .then((detailResponse) => {
              const hasCocktails = Object.keys(detailResponse.data._links).some((key) =>
                key.startsWith("cocktail-")
              );
              return { ...client, hasCocktails };
            });
        });

        Promise.all(clientPromises).then((clientsWithCocktails) => {
          this.clients = clientsWithCocktails;
        });
      })
      .catch((error) => {
        console.error("Erreur lors de la récupération des clients:", error);
      });
  },
  methods: {
    toggleDetails(clientId) {
      if (this.selectedClientId === clientId) {
        // Si déjà sélectionné, on referme les détails
        this.selectedClientId = null;
        this.clientDetails = null;
        this.cocktails = [];
        this.resetCocktailDetails();
      } else {
        // Charger les détails d'un nouveau client
        this.selectedClientId = clientId;
        this.resetCocktailDetails();
        this.getClientDetails(clientId);
      }
    },
    getClientDetails(clientId) {
      axios
        .get(`http://localhost:8080/clients/${clientId}`)
        .then((response) => {
          this.clientDetails = response.data;

          // Charger la liste des cocktails du client
          const cocktailLinks = Object.values(response.data._links).filter(
            (link) => link.href.includes("/cocktails/")
          );

          if (cocktailLinks.length > 0) {
            this.getCocktails(cocktailLinks);
          } else {
            this.cocktails = [];
          }
        })
        .catch((error) => {
          console.error("Erreur lors de la récupération des détails du client:", error);
        });
    },
    getCocktails(links) {
      // Charger tous les cocktails à partir des liens
      const cocktailRequests = links.map((link) =>
        axios.get(link.href).then((response) => response.data)
      );

      Promise.all(cocktailRequests)
        .then((cocktailResponses) => {
          this.cocktails = cocktailResponses;
        })
        .catch((error) => {
          console.error("Erreur lors de la récupération des cocktails:", error);
        });
    },
    getCocktailDetails(cocktailId) {
      axios
        .get(`http://localhost:8080/cocktails/${cocktailId}`)
        .then((response) => {
          this.selectedCocktailId = cocktailId;
          this.cocktailDetails = response.data;
        })
        .catch((error) => {
          console.error("Erreur lors de la récupération des détails du cocktail:", error);
        });
    },
    resetCocktailDetails() {
      // Réinitialiser les détails des cocktails
      this.selectedCocktailId = null;
      this.cocktailDetails = null;
    },
  },
};
</script>
