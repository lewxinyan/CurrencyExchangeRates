<template>
  <v-container>
    <div class="text-center text-h4 font-weight-bold">Currency Exchange Rates</div>
    <br><br>
    <v-row>
      <v-col cols="6">
        <CurrencySelect v-model="selectedOption1" label="From Currency" id="from-currency"></CurrencySelect>
      </v-col>
      <v-col cols="6">
        <CurrencySelect v-model="selectedOption2" label="To Currency" id="to-currency"></CurrencySelect>
      </v-col>
    </v-row>

    <v-row>
      <v-col cols="6">
        <DatePicker v-model="fromDate" label="From Date" :max="maxDate"></DatePicker>
      </v-col>
      <v-col cols="6">
        <DatePicker v-model="toDate" label="To Date" :min="fromDate" :max="maxDate"></DatePicker>
      </v-col>
    </v-row>

    <v-btn @click="handleSelection" color="primary" class="mt-4 mb-5" :disabled="!selectedOption1 || !fromDate || !toDate">Submit</v-btn>

    <ExchangeRateResults
      :showResults="showResults"
      :loading="loading"
      :error="error"
      :base="base"
      :selectedCountry="selectedCountry"
      :chartData="chartData"
      :start_date="start_date"
      :end_date="end_date"
      :rates="rates"
    />
  </v-container>
</template>

<script>
import { ref } from 'vue';
import { useExchangeRateStore } from '@/stores/exchangeRateStore';
import { useDateFormatter } from '@/composables/useDateFormatter';
import CurrencySelect from '@/components/CurrencySelect.vue';
import DatePicker from '@/components/DatePicker.vue';
import ExchangeRateResults from '@/components/ExchangeRateResults.vue';

export default {
  name: 'CurrencyExchangeRates',
  components: { CurrencySelect, DatePicker, ExchangeRateResults},

  setup() {
    const exchangeRateStore = useExchangeRateStore();
    const fromDate = ref(null);
    const toDate = ref(null);
    const { formatDateToReadable, maxDate } = useDateFormatter();

    return {
      fromDate,
      toDate,
      maxDate,
      exchangeRateStore,
      formatDateToReadable,
    };
  },
  data() {
    return {
      selectedOption1: {code: 'MYR', name: 'Malaysian Ringgit'},
      selectedOption2: {code: 'USD', name: 'US Dollar'},
      showResults: false,
      chartData: { labels:[], datasets:[]},
    }
  },
  computed: {
    base() { return this.exchangeRateStore.base;},
    start_date() { return this.exchangeRateStore.start_date;},
    end_date() { return this.exchangeRateStore.end_date;},
    rates() { return this.exchangeRateStore.rates;},
    loading() { return this.exchangeRateStore.loading;},
    error() { return this.exchangeRateStore.error;},
    selectedCountry() { return this.exchangeRateStore.selectedCountry;},
  },
  methods: {
    updateChartData() {
      if(this.rates && this.selectedCountry) {
        const dates = Object.keys(this.rates);
        const rateValues = dates.map(date =>
          Number(this.rates[date][this.selectedCountry].toFixed(5))
        );

        this.chartData = {
          labels: dates.map(date => this.formatDateToReadable(date)),
          datasets: [{
            label: `${this.selectedOption1.code} to ${this.selectedCountry} Exchange Rate`,
            backgroundColor: 'rgba(75, 192, 192, 0.2)',
            borderColor: 'rgba(75, 192, 192, 1)',
            data: rateValues
          }]
        };
      } else {
        this.chartData = { labels: [], datasets: []};
      }
    },
    async handleSelection() {
      if(this.selectedOption1.code === this.selectedOption2.code) {
        alert( "Please select different country for both options.");
        return;
      }
      if(this.fromDate && this.toDate) {
        this.showResults = true;
        await this.exchangeRateStore.fetchExchangeRates(
          this.selectedOption1.code,
          this.selectedOption2.code,
          this.fromDate,
          this.toDate
        );
        if(this.exchangeRateStore.error) {
          alert(`Error fetching exchange rates: ${this.exchangeRateStore.error}`);
          return;
        }
        this.updateChartData();
      }
    },
  },
  watch: {
    fromDate(newDate) {
      if(this.toDate && new Date(this.toDate) < new Date(newDate)) {
        this.toDate = newDate;
        console.log("Auto adjust the date.");
      }
    },
  },
}
</script>
