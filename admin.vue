<template>
  <div class="admin-container">
    <h1>Gestion des Motos & Horaires</h1>

    <!-- Formulaire d'ajout -->
    <form class="form" @submit.prevent="addVehicule">
      <input v-model="newVehicule.nom" placeholder="Nom du véhicule" required />
      <input v-model="newVehicule.image" placeholder="URL de l'image" required />
      <input v-model="newVehicule.description" placeholder="Description" required />
      <input v-model.number="newVehicule.prix" type="number" placeholder="Prix" required />
      <button type="submit">Ajouter</button>
    </form>

    <!-- Liste des véhicules -->
    <div class="vehicule-grid">
      <div v-for="vehicule in vehicules" :key="vehicule._id" class="vehicule-card">
        <img :src="vehicule.image" alt="moto" />
        <h3>{{ vehicule.nom }}</h3>
        <p>{{ vehicule.description }}</p>
        <strong>{{ vehicule.prix }} €</strong>

        <div class="horaires">
          <h4>Horaires disponibles :</h4>
          <ul>
            <li v-for="(h, index) in vehicule.horaires || []" :key="index">{{ h }}</li>
          </ul>
          <input v-model="newHoraire[vehicule._id]" placeholder="Ajouter un horaire" />
          <button @click="addHoraire(vehicule)">Ajouter l'horaire</button>
        </div>

        <div class="actions">
          <button @click="editVehicule(vehicule)">✏️ Modifier</button>
          <button @click="deleteVehicule(vehicule._id)">🗑️ Supprimer</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      vehicules: [],
      newVehicule: {
        nom: "",
        image: "",
        description: "",
        prix: 0,
        horaires: [],
      },
      newHoraire: {}, // horaire par véhicule (clé = ID)
    };
  },
  created() {
    this.fetchVehicules();
  },
  methods: {
    async fetchVehicules() {
      const res = await fetch("http://localhost:3000/vehicule");
      this.vehicules = await res.json();
    },
    async addVehicule() {
      await fetch("http://localhost:3000/vehicule", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(this.newVehicule),
      });
      this.newVehicule = { nom: "", image: "", description: "", prix: 0, horaires: [] };
      this.fetchVehicules();
    },
    async deleteVehicule(id) {
      await fetch(`http://localhost:3000/vehicule/${id}`, { method: "DELETE" });
      this.fetchVehicules();
    },
    editVehicule(vehicule) {
      this.newVehicule = { ...vehicule };
    },
    async addHoraire(vehicule) {
      const horaire = this.newHoraire[vehicule._id];
      if (!horaire) return;

      // Ajouter le nouvel horaire localement
      const updatedVehicule = {
        ...vehicule,
        horaires: [...(vehicule.horaires || []), horaire],
      };

      await fetch(`http://localhost:3000/vehicule/${vehicule._id}`, {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(updatedVehicule),
      });

      this.newHoraire[vehicule._id] = "";
      this.fetchVehicules();
    },
  },
};
</script>

<style scoped>
.admin-container {
  padding: 30px;
  font-family: 'Segoe UI', sans-serif;
  background-color: #000;
  color: #fff;
}

h1 {
  text-align: center;
  margin-bottom: 30px;
  color: #f4b400;
}

.form {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  justify-content: center;
  margin-bottom: 40px;
}

.form input {
  padding: 10px;
  border-radius: 5px;
  border: none;
  width: 200px;
}

button {
  background-color: #f4b400;
  color: black;
  padding: 10px 15px;
  font-weight: bold;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

button:hover {
  background-color: #ffd700;
}

.vehicule-grid {
  display: grid;
  gap: 20px;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
}

.vehicule-card {
  background-color: #111;
  padding: 15px;
  border-radius: 8px;
  text-align: center;
  border: 1px solid #333;
}

.vehicule-card img {
  max-width: 100%;
  height: 180px;
  object-fit: cover;
  border-radius: 5px;
  margin-bottom: 10px;
}

.actions {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-top: 10px;
}

.horaires {
  margin: 10px 0;
  color: #ccc;
}

.horaires input {
  margin-top: 5px;
  padding: 5px;
  width: 90%;
  margin-bottom: 10px;
}
</style>
