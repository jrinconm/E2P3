<template>
  <q-page class="">
    <div class="row justify-around">
      <div class="col-1" />
      <div class="col-2 q-pa-sm q-mt-md">
        <q-input v-model="search" filled type="text" hint="Buscar">
          <template v-slot:append>
            <q-icon name="search" />
          </template>
        </q-input>
      </div>
      <div class="col-2 q-pa-sm q-mt-md">
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
      <div class="col-2 q-pa-sm q-mt-md">
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
      <div class="col-2 q-pa-sm q-mt-sm">
        <q-select
          class="col-2 q-pa-sm"
          label="Ordenar"
          transition-show="flip-up"
          transition-hide="flip-down"
          filled
          v-model="orden"
          :options="options"
        />
      </div>
      <div class="col-2 q-pa-sm q-mt-lg">
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
      <div class="col-1" />
    </div>
    <div class="row justify-around">
      <div class="col-2" />
      <div class="col-8">
        <q-list bordered separator>
          <q-item clickable v-ripple>
            <q-item-section>Single line item</q-item-section>
          </q-item>

          <q-item clickable v-ripple>
            <q-item-section>
              <q-item-label>Item with caption</q-item-label>
              <q-item-label caption>Caption</q-item-label>
            </q-item-section>
          </q-item>

          <q-item clickable v-ripple>
            <q-item-section>
              <q-item-label overline>OVERLINE</q-item-label>
              <q-item-label>Item with caption</q-item-label>
            </q-item-section>
          </q-item>
        </q-list>
      </div>
      <div class="col-2" />
    </div>
  </q-page>
</template>

<script>
// import axios from "axios";
export default {
  name: "PageIndex",
  data() {
    return {
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
  methods: {
    buscar: function() {
      this.comprobar();
      this.consultar();
    },
    comprobar: function() {},
    vaciaNoticias: function() {
      this.noticias.splice(0);
    },
    consultar: function() {
      // Creo datos de prueba
      this.dateMin = "20190101";
      this.dateMax = "20200101";
      this.search = "road";
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
          for (const documento in documentos) {
            const articulo = {
              seccion: documentos[documento].section_name,
              titular: documentos[documento].headline.main
            };
            this.noticias.push(articulo);
          }
          console.log(this.noticias);
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
