<template>
    <div class="currency-converter">
        <p>API utilizada : https://app.exchangerate-api.com/ </p>
        <p>Ultima Atualizacao de Conversao: {{ lastUpdate }}</p>
      <form @submit.prevent="convertCurrency">
        <input type="number" v-model="amount" placeholder="Valor" required />
        <select v-model="fromCurrency">
            <option v-for="(rate, currency) in currencyOptions" 
            :key="currency" 
            :value="currency"
            >
            {{ currency }}
            </option>
        </select>
        <select v-model="toCurrency">
            <option v-for="(rate, currency) in currencyOptions" 
            :key="currency" 
            :value="currency"
            >
            {{ currency }}
            </option>
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
        currencyOptions: {},
        lastUpdate: '',
      };
    },
    created() {
        this.getCurrencyOptions()
    },
    methods: {
        getRequest() {
            const apiKey = process.env.VUE_APP_API_KEY;
            const url = `https://v6.exchangerate-api.com/v6/${apiKey}/latest/${this.fromCurrency}`;
            return fetch(url);
        },
        async getCurrencyOptions() {
            try {
                const response = await this.getRequest()
                const data = await response.json()
                if (data.result === 'success') {
                this.lastUpdate = data.time_last_update_utc
                this.currencyOptions = data.conversion_rates
                } else {
                this.error = 'Erro ao buscar opções de moedas.'
                }
            } catch (err) {
                console.error(err);
                this.error = 'Erro de conexão. Tente novamente.'
            }
        },
        async convertCurrency() {
            try {
                // const response = await fetch(url);
                const data = await this.getRequest().then((response) => response.json())
                console.log(data);

                if (data.result === 'success') {
                // Obter a taxa de conversão para a moeda de destino
                const rate = data.conversion_rates[this.toCurrency]
                if (rate) {
                    this.result = (this.amount * rate).toFixed(2)
                    this.error = null;
                } else {
                    this.error = `Taxa de conversão para ${this.toCurrency} não encontrada.`
                }
                } else {
                this.error = 'Erro ao buscar taxas de câmbio.'
                }
            } catch (err) {
                console.error(err);
                this.error = 'Erro de conexão. Tente novamente.'
            }
        }
    }
}
  </script>
  
  <style scoped>
  .error {
    color: red;
  }
  </style>
  