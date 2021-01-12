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
            <!-- Este mensaje lo obtengo al generar el error en la funcion-->
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
          <!-- La mascara obliga a poner 8 numeros-->
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
              <!-- Aquí me he tirado ya el moco -->
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
        <!-- Ya que he sacado las paginas de las noticias... -->
        <div class="q-pa-lg flex flex-center">
          <q-pagination
            v-show="cantidadnoticias"
            v-model="pagina"
            :max="maxpaginas"
            :input="true"
            :direction-links="true"
            :boundary-links="true"
            icon-first="skip_previous"
            icon-last="skip_next"
            icon-prev="fast_rewind"
            icon-next="fast_forward"
          >
          </q-pagination>
        </div>
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
      // Hits y pagina es para buscar más de las primeras 10 noticias
      // Hay un maximo de 1000 noticias
      hits: 0,
      pagina: 0,
      // Los campos necesarios
      search: "",
      dateMin: "",
      dateMax: "",
      orden: null,
      options: ["primero más modernas", "primero más antiguas"],
      urlApi: "https://api.nytimes.com/svc/search/v2/articlesearch.json",
      authKey: "gfgrkaJ83razF1eatts3ekuYROg4JICi",
      noticias: []
    }
  },
  computed: {
    // Obtengo la cantidad de noticias de la respuesta
    cantidadnoticias: function (){
      return this.hits
    },
    // Son 10 noticias por pagina, hasta un maximo de 100 paginas
    maxpaginas: function (){
      return (this.hits/10 > 100 ? 100 : this.hits/10)
    },
  },
  // Vigilo cuando cambian pagina
  watch: {
    pagina: function(val) {
      this.pagina = val;
      this.consultar()
      }
    },
    // Al hacer click en la noticia abre el navegador
  methods: {
    visitar: function(url){
      openURL(url)
    },
    // No es necesaria, pero me sirve para depurar
    // Dependiendo la comprobacion realiza la consulta
    buscar: function() {
      if (this.comprobar()) {
        this.consultar();
      } else {
        /* No lo ejecuto ya
        console.log("Comprobar mal") */
      }
    },
    // Realiza las comprobaciones necesarias
    comprobar: function() {
      let error = false;
      const campos = [
        {clave: "search",texto: "El campo buscar no puede estar vacio"},
      {clave: "dateMin",texto: "La fecha minima no puede estar vacía"},
      {clave: "dateMax",texto: "La fecha maxima no puede estar vacía"},
      {clave: "orden",texto: "Debes elegir un tipo de ordenación"}
      ];
      // Defino el patron regexp de 8 numeros como en la api
      const patronFecha = /^\d{8}$/;
      // Compruebo la fecha minima
      if(!patronFecha.test(this.dateMin)){
          this.mensaje = "La fecha mínima de la noticia no sigue el patrón YYYYMMDD"
          this.alert = true;
          error = true;
          return false;
      }
      // Compruebo la fecha maxima, es lo mismo que la anterior, pero no me merece la pena hacer una funcion
      if(!patronFecha.test(this.dateMax)){
          this.mensaje = "La fecha máxima de la noticia no sigue el patrón YYYYMMDD"
          this.alert = true;
          error = true;
          return false;
      }
      // Compruebo que una sea mayor que la otra, aunque sean strings, sirve igual
      if(this.dateMin>this.dateMax){
          this.mensaje = "La fecha minima no puede ser superior a la fecha maxima"
          this.alert = true;
          error = true;
          return false;
      }
      // Me buscaba primero por orden y prefiero que sea por search
      campos.reverse().forEach(campo => {
        if (!this.[campo.clave]) {
          this.mensaje = campo.texto;
          this.alert = true;
          error = true;
          return false;
        }
      });
      // Si no hay errores, devuelvo true
      if(!error){
        return true
      }
    },
    // Limpia el array de noticias
    vaciaNoticias: function() {
      this.noticias.splice(0);
    },
    // Realiza la consulta y la parsea al array de noticias
    consultar: function() {
      // Creo datos de prueba
      /* this.dateMin = "20190101";
      this.dateMax = "20200101";
      this.search = "road"; */
      // Definimos el comportamiento de Axios
      this.$axios
        .get(this.urlApi, {
          params: {
            // Los parametros necesarios
            "api-key": this.authKey,
            begin_date: this.dateMin,
            end_date: this.dateMax,
            q: this.search,
            // Este es un parametro extra
            page: this.pagina,
            sort: this.orden === "primero más modernas" ? "newest" : "oldest"
          }
        })
        .then(response => {
          this.hits = response.data.response.meta.hits;
          // Hacemos cosas con la respuesta
          const documentos = response.data.response.docs;
          // Vaciamos la lista de noticias
          this.vaciaNoticias();
          // Y la rellenamos con los datos nuevos
          for (const documento in documentos) {
            const articulo = {
              seccion: documentos[documento].section_name,
              titular: documentos[documento].headline.main,
              url: documentos[documento].web_url
            };
            this.noticias.push(articulo);
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
