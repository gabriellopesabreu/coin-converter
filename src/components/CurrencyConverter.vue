<template>
    <div class="container mt-5">
        <div class="row justify-content-center">
            <div class="col-md-6">
                <div class="card">
                    <div class="card-header bg-primary text-white text-center">
                        <h3>Conversor de Moedas</h3>
                    </div>
                    <p class="p-1 text-primary-emphasis bg-primary-subtle border border-primary-subtle rounded-2 text-center">API utilizada : https://app.exchangerate-api.com/ </p>
                    <p class="text-center">Ultima Atualizacao de Conversao: {{ lastUpdated }}</p>

                    <div class="card-body">
                        <form>
                            <div class="mb-3">
                                <label for="amount" class="form-label">Valor</label>
                                <input 
                                type="number" 
                                v-model.number="amount" 
                                placeholder="Valor" 
                                required
                                class="form-control"
                                />
                            </div>

                            <div class="mb-3 d-flex align-items-center">
                                <label for="fromCurrency" class="form-label me-3" style="width: 50px;">De</label>
                                <select 
                                    v-model="fromCurrency"
                                    class="form-select me-2"
                                    style="max-width: 150px;">
                                    <option v-for="(rate, currency) in currencyOptions" 
                                    :key="currency" 
                                    :value="currency"
                                    >
                                    {{ currency }}
                                    </option>
                                </select>
                                <span class="ms-2 text-muted">
                                    {{currencyNames[fromCurrency]}}
                                </span>
                            </div>

                            <div class="mb-3 d-flex align-items-center">
                                <label for="toCurrency" class="form-label me-3" style="width: 50px;">Para</label>
                                <select 
                                    v-model="toCurrency"
                                    class="form-select me-2"
                                    style="max-width: 150px;"
                                    >
                                    <option v-for="(rate, currency) in currencyOptions" 
                                    :key="currency" 
                                    :value="currency"
                                    >
                                    {{ currency }}
                                    </option>
                                </select>
                                <span class="ms-2 text-muted">
                                    {{currencyNames[toCurrency]}}
                                </span>
                            </div>

                            <button
                            class="btn btn-primary w-100"
                            type="button"
                            @click="convertCurrency"
                            >
                            Converter
                            </button>
                        </form>
                    </div>

                    <div class="card-footer text-center">
                        <div v-if="result" class="text-success">
                            <div v-if="conversionRate" class="mt-3 text-center">
                                <p class="text-info">Taxa de conversão: 1 {{ fromCurrency }} = {{ conversionRate }} {{ toCurrency }}</p>
                            </div>
                            <div>
                                Resultado : {{ amount }} {{ fromCurrency }} = {{ result }} {{ toCurrency }}
                            </div>
                        </div>
                        <p v-if="error" class="text-danger">
                            {{ error }}
                        </p>
                    </div>
                </div>
            </div>
        </div>
    </div>
  </template>
  
  <script>
  import currencyNames from '@/assets/currencies.json';

  export default {
    data() {
      return {
        amount: 0,
        fromCurrency: 'USD',
        toCurrency: 'BRL',
        result: null,
        error: null,
        currencyOptions: {},
        conversionRate: null,
        lastUpdated: null,
        currencyNames: currencyNames,
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
                    const rawDate = data.time_last_update_utc;
                    this.lastUpdated = rawDate.split(' ').slice(0, 4).join(' ');

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

            if (this.amount <= 0) {
                this.error = "Por favor, insira um valor maior que 0 para converter.";
                return;
            }
            try {
                // const response = await fetch(url);
                const data = await this.getRequest().then((response) => response.json())
                console.log(data);

                if (data.result === 'success') {
                // Obter a taxa de conversão para a moeda de destino
                const rate = data.conversion_rates[this.toCurrency]
                this.conversionRate = rate;
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
  