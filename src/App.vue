<template>
  <v-app>

    <v-main>
      <TableVue />
    </v-main>
  </v-app>
</template>

<script>
// import HelloWorld from './components/HelloWorld';
import TableVue from './components/TableVue.vue';

export default {
  name: 'App',

  components: {
    // HelloWorld,
    TableVue
  },

  data: () => ({
    //
  }),
};
</script>
