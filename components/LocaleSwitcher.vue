// components/LocaleSwitcher.vue
<template>
  <div class="locale-switcher">
    <!-- Language Selector -->
    <div class="selector-wrapper">
      <label>{{ currentLanguage.name }}</label>
      <select v-model="selectedLanguage" @change="changeLanguage" class="select-box">
        <option v-for="(lang, code) in languages" 
                :key="code" 
                :value="code">
          {{ lang.name }} ({{ lang.native }})
        </option>
      </select>
    </div>

    <!-- Currency Selector -->
    <div class="selector-wrapper">
      <label>{{ currentCurrency.code }}</label>
      <select v-model="selectedCurrency" @change="changeCurrency" class="select-box">
        <option v-for="(curr, code) in currencies" 
                :key="code" 
                :value="code">
          {{ curr.symbol }} {{ curr.code }} - {{ curr.name }}
        </option>
      </select>
    </div>
  </div>
</template>

<script>
export default {
  name: 'LocaleSwitcher',
  
  data() {
    return {
      selectedLanguage: 'en',
      selectedCurrency: 'USD',
      
      languages: {
        'en': { name: 'English', native: 'English' },
        'es': { name: 'Spanish', native: 'Español' },
        'fr': { name: 'French', native: 'Français' },
        'de': { name: 'German', native: 'Deutsch' },
        'it': { name: 'Italian', native: 'Italiano' },
        'pt': { name: 'Portuguese', native: 'Português' },
        'ru': { name: 'Russian', native: 'Русский' },
        'zh': { name: 'Chinese', native: '中文' },
        'ja': { name: 'Japanese', native: '日本語' },
        'ko': { name: 'Korean', native: '한국어' },
        'ar': { name: 'Arabic', native: 'العربية' }
      },

      currencies: {
        'USD': { code: 'USD', symbol: '$', name: 'US Dollar' },
        'EUR': { code: 'EUR', symbol: '€', name: 'Euro' },
        'GBP': { code: 'GBP', symbol: '£', name: 'British Pound' },
        'JPY': { code: 'JPY', symbol: '¥', name: 'Japanese Yen' },
        'CNY': { code: 'CNY', symbol: '¥', name: 'Chinese Yuan' },
        'INR': { code: 'INR', symbol: '₹', name: 'Indian Rupee' },
        'RUB': { code: 'RUB', symbol: '₽', name: 'Russian Ruble' },
        'BRL': { code: 'BRL', symbol: 'R$', name: 'Brazilian Real' },
        'KRW': { code: 'KRW', symbol: '₩', name: 'South Korean Won' },
        'AUD': { code: 'AUD', symbol: 'A$', name: 'Australian Dollar' }
      },
      
      exchangeRates: {},
    }
  },

  computed: {
    currentLanguage() {
      return this.languages[this.selectedLanguage] || this.languages.en;
    },
    
    currentCurrency() {
      return this.currencies[this.selectedCurrency] || this.currencies.USD;
    }
  },

  async created() {
    await this.detectLocation();
    await this.loadExchangeRates();
    this.loadSavedPreferences();
  },

  methods: {
    async detectLocation() {
      try {
        const response = await fetch('https://ipapi.co/json/');
        const data = await response.json();
        
        // Set initial language based on country
        const countryToLanguage = {
          'US': 'en', 'GB': 'en', 'AU': 'en',
          'ES': 'es', 'MX': 'es', 'AR': 'es',
          'FR': 'fr', 'DE': 'de', 'IT': 'it',
          // Add more mappings as needed
        };
        
        // Set initial currency based on country
        const countryToCurrency = {
          'US': 'USD', 'GB': 'GBP', 'EU': 'EUR',
          'JP': 'JPY', 'CN': 'CNY', 'IN': 'INR',
          // Add more mappings as needed
        };

        this.selectedLanguage = countryToLanguage[data.country_code] || 'en';
        this.selectedCurrency = countryToCurrency[data.country_code] || 'USD';
      } catch (error) {
        console.error('Error detecting location:', error);
      }
    },

    async loadExchangeRates() {
      try {
        const response = await fetch('https://api.exchangerate-api.com/v4/latest/USD');
        const data = await response.json();
        this.exchangeRates = data.rates;
        localStorage.setItem('exchangeRates', JSON.stringify(data.rates));
      } catch (error) {
        console.error('Error loading exchange rates:', error);
        // Try to load from localStorage if API fails
        const savedRates = localStorage.getItem('exchangeRates');
        if (savedRates) {
          this.exchangeRates = JSON.parse(savedRates);
        }
      }
    },

    loadSavedPreferences() {
      // Load saved preferences from localStorage
      const savedLang = localStorage.getItem('selectedLanguage');
      const savedCurrency = localStorage.getItem('selectedCurrency');
      
      if (savedLang && this.languages[savedLang]) {
        this.selectedLanguage = savedLang;
      }
      
      if (savedCurrency && this.currencies[savedCurrency]) {
        this.selectedCurrency = savedCurrency;
      }
    },

    changeLanguage() {
      localStorage.setItem('selectedLanguage', this.selectedLanguage);
      this.$root.$emit('languageChanged', this.selectedLanguage);
      document.documentElement.lang = this.selectedLanguage;
    },

    changeCurrency() {
      localStorage.setItem('selectedCurrency', this.selectedCurrency);
      this.$root.$emit('currencyChanged', {
        currency: this.selectedCurrency,
        rates: this.exchangeRates
      });
    },

    formatPrice(amount) {
      const rate = this.exchangeRates[this.selectedCurrency] || 1;
      const converted = amount * rate;
      
      return new Intl.NumberFormat(this.selectedLanguage, {
        style: 'currency',
        currency: this.selectedCurrency
      }).format(converted);
    }
  }
}
</script>

<style scoped>
.locale-switcher {
  display: flex;
  gap: 1rem;
  padding: 1rem;
}

.selector-wrapper {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.select-box {
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  min-width: 200px;
}

label {
  font-weight: 500;
  color: #666;
}
</style>