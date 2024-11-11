// components/PriceFormatter.vue
<template>
  <span class="price">{{ formattedPrice }}</span>
</template>

<script>
export default {
  name: 'PriceFormatter',
  
  props: {
    amount: {
      type: Number,
      required: true
    }
  },
  
  data() {
    return {
      currentCurrency: 'USD',
      exchangeRates: {}
    }
  },
  
  computed: {
    formattedPrice() {
      const rate = this.exchangeRates[this.currentCurrency] || 1;
      const converted = this.amount * rate;
      
      return new Intl.NumberFormat(document.documentElement.lang || 'en', {
        style: 'currency',
        currency: this.currentCurrency
      }).format(converted);
    }
  },
  
  created() {
    // Load saved currency and rates
    const savedCurrency = localStorage.getItem('selectedCurrency');
    const savedRates = localStorage.getItem('exchangeRates');
    
    if (savedCurrency) {
      this.currentCurrency = savedCurrency;
    }
    
    if (savedRates) {
      this.exchangeRates = JSON.parse(savedRates);
    }
    
    // Listen for currency changes
    this.$root.$on('currencyChanged', ({ currency, rates }) => {
      this.currentCurrency = currency;
      this.exchangeRates = rates;
    });
  }
}
</script>