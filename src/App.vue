<template>
  <v-app>
    <v-main>
      <Header />
      <ParamInput />
    </v-main>
  </v-app>
</template>

<script>
import Header from './components/Header.vue'
import ParamInput from './components/ParamInput.vue'

export default {
  name: 'App',

  components: {
    Header,
    ParamInput
  },

  data: () => ({
    //
  }),
};
</script>
