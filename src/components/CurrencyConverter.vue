<template>
    <div class="currency-converter">
      <form @submit.prevent="convertCurrency">
        <input type="number" v-model="amount" placeholder="Valor" required />
        <select v-model="fromCurrency">
          <option value="USD">USD</option>
          <option value="EUR">EUR</option>
          <option value="BRL">BRL</option>
        </select>
        <select v-model="toCurrency">
          <option value="USD">USD</option>
          <option value="EUR">EUR</option>
          <option value="BRL">BRL</option>
        </select>
        <button type="submit">Converter</button>
      </form>
      <p v-if="result">{{ amount }} {{ fromCurrency }} = {{ result }} {{ toCurrency }}</p>
      <p v-if="error" class="error">{{ error }}</p>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        amount: '',
        fromCurrency: 'USD',
        toCurrency: 'BRL',
        result: null,
        error: null,
      };
    },
    methods: {
      async convertCurrency() {
        const apiKey = process.env.VUE_APP_API_KEY;
        const url = `https://v6.exchangerate-api.com/v6/${apiKey}/latest/${this.toCurrency}`;
  
        try {
          const response = await fetch(url);
          const data = await response.json();
          console.log(data)
          if (data.result === 'success') {
            this.result = (this.amount * data.conversion_rate).toFixed(2);
            this.error = null;
          } else {
            this.error = 'Erro ao buscar taxas de câmbio.';
          }
        } catch (err) {
          this.error = 'Erro de conexão. Tente novamente.';
        }
      },
    },
  };
  </script>
  
  <style scoped>
  .error {
    color: red;
  }
  </style>
  