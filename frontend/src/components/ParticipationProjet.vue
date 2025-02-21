<script setup>
import { ref, onMounted, computed } from "vue";
import axios from 'axios';

// Objet participation lié à une personne et un projet
console.log("bjr");
const nouvelleParticipation = ref({
  personneId: "",
  projetId: "",
  role: "",
  pourcentage: "",
});

// Listes des données
const personnes = ref([]);
const projets = ref([]);

// Liste des projets non terminés
/*const projetsNonTermines = computed(() => {
  return projets.value.filter(projet => !projet.fin);
});*/

// Messages d'état
const message = ref("");
const successMessage = ref(false);
const errorMessage = ref(false);

// Vérification si le formulaire est valide
const formValide = computed(() => {
  return (
    nouvelleParticipation.value.personneId &&
    nouvelleParticipation.value.projetId &&
    nouvelleParticipation.value.role &&
    nouvelleParticipation.value.pourcentage !== ""
  );
});

// Chargement des données depuis l'API
const fetchData = async () => {
  try {
    const [resPersonnes, resProjets] = await Promise.all([
      axios.get("http://localhost:8989/api/personnes"),
      axios.get("http://localhost:8989/api/projets")
    ]);
    personnes.value = resPersonnes.data._embedded.personnes;

    let resultProjet = resProjets.data._embedded.projets

    for (let projet of resultProjet){
      if (projet.fin == null){
        projets.value.push(projet);
      }
    }

    //projets.value = resProjets.data;

  } catch (error) {
    console.error("Erreur lors du chargement des données", error);
    message.value = "Erreur lors du chargement des données";
  }
};

// Soumission du formulaire
const submitParticipation = async () => {
  try {

    // Conversion du pourcentage en valeur décimale
   /* let participationData = { ...nouvelleParticipation.value };
    participationData.pourcentage = participationData.pourcentage / 100;

    await axios.post("/api/participation", participationData);*/


    await axios.post("/api/participations", nouvelleParticipation.value);
    message.value = "Participation enregistrée avec succès !";
    successMessage.value = true;
    errorMessage.value = false;

    // Réinitialisation du formulaire
    nouvelleParticipation.value = { personneId: "", projetId: "", role: "", pourcentage: "" };
  } catch (error) {
    //message.value = "Erreur lors de l'enregistrement.";
    if (error.response && error.response.data && error.response.data) {
      message.value = error.response.data.message; // Récupération du message d'erreur du backend
    } else {
      message.value = "Erreur lors de l'enregistrement.";
    }
    successMessage.value = false;
    errorMessage.value = true;
  }
};

onMounted(fetchData);

</script>

<template>
  <h1>Enregistrer une participation</h1>

  <form @submit.prevent="submitParticipation">
    <label for="personneId">Personne</label>
    <select id="personneId" v-model="nouvelleParticipation.personneId" required>
      <option disabled value="">Choisissez une personne</option>
      <option v-for="personne in personnes" :key="personne.id" :value="personne.id">{{ personne.nom}}</option>
    </select>

    <label for="projetId">Projet</label>
   <select id="projetId" v-model="nouvelleParticipation.projetId" required>
      <option disabled value="">Choisissez un projet</option>
      <option v-for="projet in projets" :key="projet.id" :value="projet.id">{{ projet.nom}}</option>
    </select>


    <label for="role">Rôle</label>
    <input id="role" type="text" v-model="nouvelleParticipation.role" required/>

    <label for="pourcentage">Pourcentage</label>
    <input id="pourcentage" required type="range" v-model="nouvelleParticipation.pourcentage" min="0" max="100">
    <p>{{ nouvelleParticipation.pourcentage}}%</p>

    <button type="submit">Enregistrer</button>
  </form>

  <!-- Message d'erreur ou succès -->
  <p v-if="message" :class="{ success: successMessage, error: errorMessage }">{{ message }}</p>

</template>

<style scoped>

</style>
