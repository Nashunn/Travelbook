<template>
<!-- Card Creation Form -->
    <section class="container">
        <h2 class="center-align specialTitle">Création d'une fiche</h2>
        <div class="section-deco center-align">v</div>
        <div class="deco-line col s12 m12 center-align"></div>

        <div id="card-creation-section" class="row">
            <div class="creation-section col s12 m10 offset-m1">
                <h3 class="specialTitle">Image de couverture</h3>
                <vue-dropzone
                    ref="myVueDropzone" id="dropzone"
                    class="tb-dropzone center-align"
                    :options="dropzoneOptions"
                    @vdropzone-complete="afterComplete" style=""
                    v-on:vdropzone-removed-file="removeThisFile"

                />

                <div class="col s12 padding-left-0">
                    <h3 class="specialTitle">Titre</h3>
                    <input class="tb-input tb-input-100" v-model="infos.title" placeholder="Votre titre"/>
                </div>

                <div class="col s12 padding-left-0">
                    <h3 class="specialTitle">Adresse</h3>
                    <gmap-autocomplete
                        class="tb-input tb-input-100"
                        @place_changed="setPlace"
                    />
                </div>

                <div class="col s12 padding-left-0">
                    <h3 class="specialTitle">Catégorie</h3>
                    <select class="tb-select tb-input-100" v-model="infos.category">
                        <option disabled value="">Choisissez une catégorie</option>
                        <option value="Restaurant">Restaurant</option>
                        <option value="Hotel">Hotel</option>
                        <option value="Lieu">Lieu</option>
                        <option value="Monument">Monument</option>
                    </select>
                </div>

                <div class="col s12 padding-left-0">
                    <h3 class="specialTitle">Première colonne</h3>
                    <vue-dropzone
                        ref="myVueDropzone" id="img2"
                        class="tb-dropzone"
                        :options="dropzoneOptions"
                        @vdropzone-complete="afterComplete"
                    />
                    <textarea
                        v-model="infos.para1"
                        id="textarea1" class="tb-txtarea tb-input-100"
                        placeholder="Votre texte">
                    </textarea>
                </div>

                <div class="col s12 padding-left-0">
                    <h3 class="specialTitle">Deuxième colonne</h3>
                    <vue-dropzone
                        ref="myVueDropzone" id="img3"
                        class="tb-dropzone"
                        :options="dropzoneOptions"
                        @vdropzone-complete="afterComplete"
                    />
                    <textarea
                        v-model="infos.para2"
                        id="textarea2" class="tb-txtarea tb-input-100"
                        placeholder="Votre texte">
                    </textarea>
                </div>
            </div>
        </div>
        <div class="row center-block">
            <button @click="submit()" class="tb-btn btn-red create-card-btn col s4 offset-s4 m4 offset-m4">
                Créer
            </button>
        </div>
    </section>
<!-- End Card Creation Form -->
</template>

<script>
import Vue from "vue";
import { EventBusModal } from "../events/event-modals";
import vue2Dropzone from "vue2-dropzone";
import "vue2-dropzone/dist/vue2Dropzone.css";
import { HTTP } from "./../http/http-base";
import * as VueGoogleMaps from "vue2-google-maps";
import router from "./../router";
import VueSweetalert2 from 'vue-sweetalert2';

Vue.use(VueGoogleMaps, {
  load: {
    //@todo: need to be erased in the final git
    key: "AIzaSyAnrCMvYZ_puf67n8Xj9tU_9LNgQgpGo90",
    libraries: "places"
  }
});

Vue.use(VueSweetalert2);

export default {
  components: {
    vueDropzone: vue2Dropzone,
  },
  data() {
    return {
      dropzoneOptions: {
        url: "https://httpbin.org/post",
        thumbnailWidth: 150,
        maxFiles: 1,
        maxFilesize: 2,
        addRemoveLinks: true,
        headers: { "My-Awesome-Header": "header value" },
        dictDefaultMessage: "Glisser votre image dans la zone pour la télécharger",
        dictRemoveFile: "Suppression de l'image",
        acceptedFiles: ".jpeg,.jpg,.png,.svg",
        accept: function accept(file, done) {
          done();
        }
      },
      infos: {
        title: "",
        category: "",
        para1: "",
        para2: "",
        author: this.$store.state.usr.username,
        adress: "",
        pictures: []
      }
    };
  },
  methods: {
    removeThisFile (file) {
        let FileName = file.name
        this.infos.pictures = this.infos.pictures.filter( file => file.name !== FileName);
    },
    afterComplete(file) {
        this.infos.pictures.push(file); //todo : gérer le remove + push img dans l'ordre (cover, 1 et 2)
    },
    submit() {
        EventBusModal.$emit("loading-loader", true);
        HTTP.post("/card", this.infos).then(response => {
            EventBusModal.$emit("loading-loader", false);
            console.log(response);
            Vue.swal({
              type: 'info',
              title: 'Fiche créée',
              text: 'Votre fiche à été ajoutée ! '
            });
            this.$router.push("/");
        }).catch(error => {
            console.log(error.response)
            Vue.swal({
              type: 'error',
              title: 'Erreur detectée' + error.response.status,
              text: 'Une erreur a été détectée. Vérifiez le formulaire',
          });
        });
    },
    setPlace(place) {
        console.log(place.formatted_address)
      this.infos.adress = place.formatted_address;
    },
  }
};
</script>
<style scoped>

</style>
