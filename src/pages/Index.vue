<template>
  <q-page class="flex flex-center">
    <q-input v-model="search" filled type="text" hint="Buscar">
      <template v-slot:append>
        <q-icon name="search" />
      </template>
    </q-input>
    <q-input v-model="search" filled type="text" hint="Fecha mínima de la noticia">
    </q-input>
    <q-input v-model="search" filled type="text" hint="Fecha máxima">
    </q-input>
  </q-page>
</template>

<script>
export default {
  name: 'PageIndex'
}
</script>
