<template>
  <q-page class="">
    <!-- Mensaje de alerta -->
    <div class="q-pa-md q-gutter-sm">
      <q-dialog v-model="alert">
        <q-card>
          <q-card-section>
            <div class="text-h6">Alert</div>
          </q-card-section>
          <q-card-section class="q-pt-none">
            {{ mensaje }}
          </q-card-section>
          <q-card-actions align="right">
            <q-btn flat label="OK" color="primary" v-close-popup />
          </q-card-actions>
        </q-card>
      </q-dialog>
    </div>
    <!-- Barra de entrada -->
    <div class="row justify-around">
      <div class="col-1" />
      <div class="col-2 q-pa-sm q-mt-md">
        <div class="row">
          <q-input v-model="search" filled type="text" hint="Buscar">
            <template v-slot:append>
              <q-icon name="search" />
            </template>
          </q-input>
        </div>
        <div class="row">
          <q-input
            v-model="dateMin"
            filled
            type="text"
            placeholder="YYYYMMDD"
            mask="########"
            hint="Fecha mínima de la noticia"
            maxlength="8"
          >
            <template v-slot:prepend>
              <q-icon name="event" />
            </template>
          </q-input>
        </div>
        <div class="row">
          <q-input
            v-model="dateMax"
            filled
            type="text"
            placeholder="YYYYMMDD"
            mask="########"
            hint="Fecha máxima"
            maxlength="8"
          >
            <template v-slot:prepend>
              <q-icon name="event" />
            </template>
          </q-input>
        </div>
        <div class="row">
          <q-select
            class="col-12"
            label="Ordenar"
            transition-show="flip-up"
            transition-hide="flip-down"
            filled
            v-model="orden"
            :options="options"
          />
        </div>
        <div class="row q-pa-sm">
          <q-btn
            @click="buscar"
            class="glossy"
            icon="search"
            icon-right="send"
            rounded
            color="deep-orange"
            label="Buscar"
          />
        </div>
      </div>
      <!-- Parte 2 de la pantalla panel grande -->
      <div class="col-7 q-ml-sm q-mr-md">
        <q-list bordered separator>
          <q-item>
            <q-item-section>
              <q-item-label>Listado de noticias</q-item-label>
              <q-item-label caption
                >Pulse en una noticia para visitar</q-item-label
              >
            </q-item-section>

            <q-item-section side top>
              <q-item-label caption>
                {{ cantidadnoticias }} noticias</q-item-label
              >
            </q-item-section>
          </q-item>
          <template v-for="(noticia, indice) in noticias">
            <q-item :key="indice" clickable v-ripple>
              <q-item-section @click="visitar(noticia.url)">
                <q-item-label>{{ noticia.titular }}</q-item-label>
                <q-item-label caption>{{ noticia.seccion }}</q-item-label>
              </q-item-section>
            </q-item>
          </template>
        </q-list>
      </div>
      <div class="col-1" />
    </div>
  </q-page>
</template>

<script>
import { openURL } from 'quasar'
export default {
  name: "PageIndex",
  data() {
    return {
      alert: false,
      mensaje: "",
      search: "",
      dateMin: "",
      dateMax: "",
      orden: null,
      options: ["primero más modernas", "primero más antiguas"],
      urlApi: "https://api.nytimes.com/svc/search/v2/articlesearch.json",
      authKey: "gfgrkaJ83razF1eatts3ekuYROg4JICi",
      noticias: []
    };
  },
  computed: {
    cantidadnoticias: function (){
      return Object.keys(this.noticias).length
    }
  },
  methods: {
    visitar: function(url){
      openURL(url)
    },
    buscar: function() {
      if (this.comprobar()) {
        console.log("Comprobar correcto")
        this.consultar();
      } else {
        console.log("Comprobar mal")
      }
    },
    comprobar: function() {
      let error = false;
      const campos = [
        {clave: "search",texto: "El campo buscar no puede estar vacio"},
      {clave: "dateMin",texto: "La fecha minima no puede estar vacía"},
      {clave: "dateMax",texto: "La fecha maxima no puede estar vacía"},
      {clave: "orden",texto: "Debes elegir un tipo de ordenación"}
      ];
      // Me buscaba primero por orden y prefiero que sea por search
      campos.reverse().forEach(campo => {
        if (!this.[campo.clave]) {
          this.mensaje = campo.texto;
          this.alert = true;
          error = true;
          return false;
        }
      });
      if(!error){
        return true
      }
    },
    vaciaNoticias: function() {
      this.noticias.splice(0);
    },
    consultar: function() {
      // Creo datos de prueba
      /* this.dateMin = "20190101";
      this.dateMax = "20200101";
      this.search = "road"; */
      // Definimos el comportamiento de Axios
      this.$axios
        .get(this.urlApi, {
          params: {
            "api-key": this.authKey,
            begin_date: this.dateMin,
            end_date: this.dateMax,
            q: this.search,
            sort: this.orden === "primero más modernas" ? "newest" : "oldest"
          }
        })
        .then(response => {
          // Hacemos cosas con la respuesta
          const documentos = response.data.response.docs;
          // Vaciamos la lista de noticias
          this.vaciaNoticias();
          for (const documento in documentos) {
            console.log(documentos[documento]);
            const articulo = {
              seccion: documentos[documento].section_name,
              titular: documentos[documento].headline.main,
              url: documentos[documento].web_url
            };
            this.noticias.push(articulo);
            console.log(this.noticias)
          }
        })
        // En caso de error, mostramos el error para facilitar depuración
        .catch(error => {
          console.log("-----error-------");
          console.log(error);
        });
    }
  }
};
</script>
