<template>
    <div class="content-header">
      <div class="container-fluid">
        <div class="row mb-2">
          <div class="col-sm-6" >
            <h1 class="m-0" style="font-weight: 500 !important;">Les valeurs des paiements</h1>
          </div>
          <div class="col-sm-6">
            <ol class="breadcrumb float-sm-right"></ol>
          </div>
        </div>
      </div>
    </div>

    <div class="content">
      <div class="container-fluid">
        <div class="container" style="overflow: auto !important; height: calc(100vh - 176px) !important;">
         <table class="table table-striped">
  <thead>
    <tr> <br> <th :colspan="TotalMat" style="text-align: center !important;">Nombre de matières</th>
</tr>
    <tr>

        <!-- <th></th> -->
      <th >Niveaux</th>


      <!-- Utilisez NbrMat pour déterminer le nombre de colonnes -->
      <template v-for="col in NbrMat" :key="col">
        <th v-if="col !== TotalMat + 1" :key="col">{{ col }}</th>
      </template>
    </tr>
  </thead>
  <tbody>

    <!-- <td rowspan="9">hdhdh</td> -->
    <tr v-for="(niveau, niveauIndex) in niveaux" :key="niveau.id">
      <td>{{ niveau }}</td>
      <template v-for="(nbr, nbrIndex) in NbrMat" :key="nbrIndex">
        <!-- Assurez-vous que les valeurs existent avant de les afficher -->
        <td v-if="(niveau == '6éme primaire' || niveau == '5éme primaire') && (nbrIndex != 1) && (nbrIndex !== 0 )">

        </td>
        <td v-else-if="nbrIndex !== 0 && valeursPaiements[niveauIndex][nbrIndex] !== undefined">
          {{ valeursPaiements[niveauIndex][nbrIndex] }} dh
        </td>
        <td v-else-if="nbrIndex !== 0 && valeursPaiements[niveauIndex][nbrIndex] === undefined">
          <!-- Si la valeur est indéfinie, affichez un espace vide -->
          &nbsp;
        </td>

      </template>
      <td>
                <a v-if="IsAdmin" @click.prevent="editValeursPaiment(niveau,valeursPaiements[niveauIndex])" class="btn btn-primary btn-sm">
  <i class="fa fa-edit"></i>
</a>  </td>
    </tr>
  </tbody>
</table>

        </div>

      </div>

    </div>









 <!-- Modal pour modifier les valeurs de paiements -->
<div class="modal fade" id="userFormModal" tabindex="-1" role="dialog" aria-labelledby="userFormModalLabel" aria-hidden="true">
  <div class="modal-dialog" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="userFormModalLabel">
          <span v-if="NivToEdit === '6éme primaire' || NivToEdit === '5éme primaire'">Modifier la valeur de paiement pour {{ NivToEdit }}</span>
          <span v-else>Modifier les valeurs de paiement pour {{ NivToEdit }}</span>
        </h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <form>
    <div class="modal-body">
      <template v-if="NivToEdit === '6éme primaire' || NivToEdit === '5éme primaire'">
        <div v-for="(nbr, nbrIndex) in NbrMat" :key="nbrIndex">
          <div v-if="nbr === 1" class="form-group">
            <label :for="'Valeur_paiment_' + nbrIndex">Valeur de paiement pour {{ NivToEdit }}</label>
            <input
              v-model="formValues[nbrIndex]"
              type="number"
              class="form-control"
              :id="'Valeur_paiment_' + nbrIndex"
              placeholder="Entrer la nouvelle valeur pour ce niveau"
              required
            />
            <span class="invalid-feedback">{{ errors[nbrIndex] }}</span>
          </div>
        </div>
      </template>

      <template v-else>
        <div v-for="(nbr, nbrIndex) in NbrMat" :key="nbrIndex">
          <div v-if="nbr !== TotalMat + 1" class="form-group">
            <label :for="'Valeur_paiment_' + nbrIndex">Valeur de paiement pour {{ nbr }} {{ nbr > 1 ? 'matières' : 'matière' }}</label>
            <input
              v-model="formValues[nbrIndex]"
              type="number"
              class="form-control"
              :id="'Valeur_paiment_' + nbrIndex"
              placeholder="Entrer la nouvelle valeur pour ce nombre de matières"
              required
            />
            <span class="invalid-feedback">{{ errors[nbrIndex] }}</span>
          </div>
        </div>
      </template>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-secondary" @click="resetForm" data-dismiss="modal">Annuler</button>
      <button type="button" class="btn btn-primary" @click.prevent="updateValeursPaiment">Enregistrer</button>
    </div>
  </form>
    </div>
  </div>
</div>



</template>

  <script setup>
  import axios from 'axios';
  import { ref, reactive, onMounted } from 'vue'; // Utilisez cette ligne pour inclure les fonctions nécessaires
  import { useToastr } from '../../toastr.js';

const toastr = useToastr();


  const NbrMat = ref([]);
  const niveaux = ref([]);
  const valeursPaiements = ref([]);
  const errors = reactive({});

  const updateValuesPeriodically = () => {
  setInterval(() => {
    getValeurPaiement(); // Cette fonction récupère les nouvelles valeurs depuis le backend
  }, 50000000); // Met à jour toutes les 5 secondes, tu peux ajuster cette valeur selon tes besoins
};




  const resetForm = () => {
    // Réinitialiser les valeurs du formulaire
    for (const key in formValues) {
      formValues[key] = '';
    }
  };

  const NivToEdit = ref() ;



  // Fonction pour récupérer les valeurs à modifier
  const editValeursPaiment = (niveau, index) => {
  $('#userFormModal').modal('show');
  NivToEdit.value = niveau;

  // Réinitialiser les valeurs du formulaire
  for (const key in formValues) {
    formValues[key] = valeursPaiements[niveau][index][key];
  }
};

const IsAdmin = ref('');

const getRole = () => {
  axios.get('/api/getRole')
    .then((response) => {
      IsAdmin.value = response.data;


    }

    )
    .catch((error) => {
      console.error('Erreur lors de la récupération de role:', error);
    });
};




const updateValeursPaiment = () => {
  const dataToUpdate = {
    NivToEdit: NivToEdit.value, // Inclure NivToEdit.value dans les données à envoyer
    formValues: Object.keys(formValues).map((key) => ({
      id: formValues[key].id,
      Valeur_paiment: formValues[key],
    })),
  };

  axios.put('/api/valeurs_paiments', dataToUpdate)
    .then((response) => {
      console.log("Response:", response.data);
      // Réinitialiser le formulaire ou effectuer d'autres actions nécessaires après l'enregistrement
      resetForm();
      $('#userFormModal').modal('hide'); // Fermer la modale après l'enregistrement
      toastr.success('Valeurs modifié avec succès !');
      getValeurPaiement();
    })
    .catch((error) => {
      console.error("Error:", error);
      // Gérer les erreurs
    });
};

const initFormValues = () => {
  // Réinitialiser les valeurs du formulaire
  for (const key in formValues) {
    formValues[key] = valeursPaiements[NivToEdit.value][key][key];
  }
};






  const cancelEdit = () => {
    // Réinitialiser le formulaire et quitter le mode d'édition
    resetForm();
    editing.value = false;
  };

  const formValues = reactive({
  // Initialize formValues as an array with the length of NbrMat
  ...Array.from({ length: NbrMat.value.length - 1 }, () => ({ Valeur_paiment: '' })),
});

  const editing = ref(false);

  let TotalMat= 5;

  const getNbrMateres = () => {
    axios.get('/api/valeurs_paiments')
      .then((response) => {
        NbrMat.value = response.data;
        TotalMat = response.data - 1;

      })
      .catch((error) => {
        console.error('Erreur lors de la récupération du nombre de matières :', error);
      });
  };

  const refreshPage = () => {
  window.location.reload();
};

  const getNiveaux = () => {

    axios.get('/api/niveaux')
      .then((response) => {
        niveaux.value = response.data;

      })
      .catch((error) => {
        console.error('Erreur lors de la récupération des niveaux :', error);
      });
  };

  // Fonction pour obtenir la valeur de paiement pour une combinaison niveau/nombre de matières
  const getValeurPaiement = () => {
  return axios.get(`/api/valeurs_paiment`)
    .then((response) => {
      const rawData = response.data;
      // Convertir les données en un objet JavaScript
      const processedData = {};

      for (const key in rawData) {
        processedData[key] = { ...rawData[key] };
      }

      valeursPaiements.value = processedData;
    })
    .catch((error) => {
      console.error('Erreur lors de la récupération des valeurs :', error);
    });
};






  onMounted(() => {
    // window.location.reload(); // Cette ligne actualisera la page

    getNiveaux();
    getNbrMateres();
    getValeurPaiement();
      updateValuesPeriodically(); // Lance le polling pour les mises à jour régulières

      getRole();

  });

  </script>
