<template>
  <div id="app">
    <h1>Conversor de Moedas</h1>
    <CurrencyConverter />
  </div>
</template>

<script>
import CurrencyConverter from './components/CurrencyConverter.vue';

export default {
  name: 'App',
  components: {
    CurrencyConverter,
  },
};
</script>
