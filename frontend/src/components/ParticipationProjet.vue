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

    console.log(typeof nouvelleParticipation.value.pourcentage)

    await axios.post("/api/gestion/participation", nouvelleParticipation.value);


    console.log("Données envoyées :", participationData); // Debugging

    await axios.post("/api/gestion/participation", participationData);

    message.value = "Participation enregistrée avec succès !";
    successMessage.value = true;
    errorMessage.value = false;

    // Réinitialisation du formulaire
    nouvelleParticipation.value = { personneId: "", projetId: "", role: "", pourcentage: "" };
  } catch (error) {
    console.log(typeof nouvelleParticipation.value.pourcentage)
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
      <option v-for="personne in personnes" :key="personne.matricule" :value="personne.matricule">{{ personne.nom}}</option>
    </select>

    <label for="projetId">Projet</label>
   <select id="projetId" v-model="nouvelleParticipation.projetId" required>
      <option disabled value="">Choisissez un projet</option>
      <option v-for="projet in projets" :key="projet.id" :value="projet.id">{{ projet.nom}}</option>
    </select>


    <label for="role">Rôle</label>
    <input id="role" type="text" v-model="nouvelleParticipation.role" required/>

    <label for="pourcentage">Pourcentage</label>
    <input id="pourcentage" required type="range" v-model.number="nouvelleParticipation.pourcentage" min="0" max="1" step="0.01">
    <p>{{ (nouvelleParticipation.pourcentage * 100).toFixed(0) }}%</p>

    <button type="submit">Enregistrer</button>
  </form>

  <!-- Message d'erreur ou succès -->
  <p v-if="message" :class="{ success: successMessage, error: errorMessage }">{{ message }}</p>

</template>

<style scoped>

/* Mise en forme générale du formulaire */
form {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

/* Titre du formulaire */
h1 {
  text-align: center;
  color: #333;
  font-size: 2rem;
  margin-bottom: 20px;
}

/* Style des labels */
label {
  display: block;
  margin-bottom: 8px;
  font-weight: bold;
  color: #444;
}

/* Style des inputs, selects et range */
input[type="text"],
select,
input[type="range"] {
  width: 100%;
  padding: 10px;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  margin-bottom: 15px;
  box-sizing: border-box;
  background-color: #fff;
}

/* Hover effect for inputs */
input[type="text"]:hover,
select:hover,
input[type="range"]:hover {
  border-color: #888;
}

/* Style pour le range input */
input[type="range"] {
  -webkit-appearance: none;
  appearance: none;
  height: 10px;
  border-radius: 5px;
  background: #ddd;
  outline: none;
  transition: background 0.3s;
}

/* Style de la poignée du range input */
input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #4caf50;
  cursor: pointer;
}

/* Lorsque l'on survole la poignée */
input[type="range"]:hover::-webkit-slider-thumb {
  background: #45a049;
}

/* Affichage du pourcentage */
p {
  text-align: center;
  font-size: 1.2rem;
  color: #333;
  font-weight: 500;
  margin-top: 5px;
}

/* Bouton de soumission */
button {
  width: 100%;
  padding: 12px;
  font-size: 1.1rem;
  color: #fff;
  background-color: #4caf50;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

/* Change de couleur du bouton au survol */
button:hover {
  background-color: #45a049;
}

/* Messages d'état (succès / erreur) */
p.success {
  color: #4caf50;
  text-align: center;
  font-weight: bold;
}

p.error {
  color: #f44336;
  text-align: center;
  font-weight: bold;
}

</style>
