<template>
  <div ref="headerWrapper">
    <header :class="{ 'no-banner': (topBannerLoaded && isTopBannerClosed) || !isPublic }">
      <banner v-if="!isTopBannerClosed" class="top-banner" :banner="topBanner" @close="topBannerClosed" />
      
      <!-- Top Bar -->
      <div class="top-bar">
        <div class="container-fluid">
          <div class="top-bar-content">
            <div class="top-left">
              <dropdown 
                v-if="Object.keys(languages).length > 1" 
                :selected-key="currentLanguage.code" 
                :options="languages"
                key-name="name" 
                class="language-selector" 
                @clicked="selectedLanguage" 
              />
<!--               
              <a :href="`mailto:${email}`" class="contact-link">
                <i class="icon email-icon" />
                <span>{{ email }}</span>
              </a> -->

              <!-- <template v-if="phone">
                <div class="separator"></div>
                <a :href="`tel:${phone}`" class="contact-link">
                  <i class="icon phone-icon" />
                  <span>{{ phone }}</span>
                </a>
              </template> -->

 
              <template>
  <div>
    <!-- Total User Count Display -->
    <span class="total-user-count">Total Users: 2</span>
  </div>
</template>
<div>
   
    <a href="https://wa.me/+971529629717" target="_blank" class="whatsapp-float">
      <i class="fab fa-whatsapp"></i>
    </a>
  </div>



              <template v-if="userCount">
                <div class="separator"></div>
                <div class="users-count">
                  <i class="icon users-icon" />
                  <span>{{ typeof userCount === 'number' ? userCount.toLocaleString() : userCount }} {{ $t('header.totalUsers') }}</span>
                </div>
              </template>
            </div>

            <div class="top-right">
              <template v-if="!isLoggedIn">
                <nuxt-link to="/login" class="auth-link">
                  <i class="icon login-icon" />
                  <span>{{ $t('header.login') }}</span>
                </nuxt-link>
                <div class="separator"></div>
                <nuxt-link to="/register" class="auth-link">
                  <i class="icon register-icon" />
                  <span>{{ $t('header.register') }}</span>
                </nuxt-link>
              </template>
              <nuxt-link v-else to="/user/profile" class="auth-link">
                <i class="icon user-icon" />
                <span>{{ $t('header.profile') }}</span>
              </nuxt-link>
            </div>
          </div>
        </div>
      </div>

      <!-- Main Header -->
      <div class="header-main" :class="{ sticky: headerSticky }">
        <div class="container-fluid">
          <div class="header-content">
            <nuxt-link to="/" class="logo">
              <img 
                :src="imageURL({ 'image': site_setting.header_logo })" 
                :alt="$t('footer.siteLogo')" 
                height="40" 
                width="139"
              >
            </nuxt-link>

            <form class="search-form" @submit.prevent="search">
              <div class="search-input-wrapper">
                <input 
                  type="text" 
                  :placeholder="$t('header.searchHere')"
                  v-model="searchedText"
                  @focus="openSearchPopup"
                  @blur="blurSearchInput"
                >
                <button aria-label="submit" type="submit" class="search-button">
                  <i class="icon search-icon" />
                </button>
              </div>
              <search-popup 
                v-if="searchPopup" 
                :searched-text="searchedText" 
                @close="closeSearchPopup" 
              />
            </form>

            <div class="header-actions">
              <div class="account-dropdown" v-outside-click="closeDropdown">
                <button 
                  aria-label="account" 
                  class="account-button"
                  @click="dropdown = !dropdown"
                >
                  <span>{{ $t('header.account') }}</span>
                  <i class="icon arrow-down" :class="{ rotated: dropdown }" />
                </button>
                
                <div class="dropdown-menu" :class="{ active: dropdown }">
                  <nuxt-link to="/user/orders" class="dropdown-item">
                    <i class="icon order-icon" />
                    {{ $t('header.orders') }}
                  </nuxt-link>
                  <nuxt-link to="/user/wishlists" class="dropdown-item">
                    <i class="icon wishlist-icon" />
                    {{ $t('header.wishList') }}
                  </nuxt-link>
                  <nuxt-link to="/user/compared" class="dropdown-item">
                    <i class="icon compare-icon" />
                    {{ $t('header.comparedList') }}
                  </nuxt-link>
                  <nuxt-link to="/user/vouchers" class="dropdown-item">
                    <i class="icon voucher-icon" />
                    {{ $t('header.vouchers') }}
                  </nuxt-link>
                  <button 
                    v-show="isLoggedIn" 
                    class="dropdown-item"
                    @click.prevent="loggingOut"
                  >
                    <i class="icon logout-icon" />
                    {{ $t('header.logout') }}
                  </button>
                </div>
              </div>

              <nuxt-link to="/cart" class="cart-button">
                <span v-if="cartCount" class="cart-badge">
                  {{ cartCount }}
                </span>
                <i class="icon cart-icon" />
                <span class="cart-text">{{ $t('header.cart') }}</span>
              </nuxt-link>
            </div>
          </div>
        </div>
      </div>

      <!-- Navigation Menu -->
      <nav class="nav-menu">
        <div class="container-fluid">
          <div class="nav-content">
            <div class="nav-left">
              <nuxt-link 
                v-for="(item, index) in headerLeft" 
                :key="index" 
                :to="getUrl(item)"
                class="nav-link"
              >
                {{ getTitle(item) }}
              </nuxt-link>
            </div>
            <div class="nav-right">
              <nuxt-link 
                v-for="(item, index) in headerRight" 
                :key="index" 
                :to="getUrl(item)"
                class="nav-link"
              >
                {{ getTitle(item) }}
              </nuxt-link>
            </div>
          </div>
        </div>
      </nav>
    </header>
  </div>
</template>

<script>
import outsideClick from '~/directives/outside-click'
import util from '~/mixin/util'
import { mapGetters, mapActions } from 'vuex'
import SearchPopup from "../../components/SearchPopup";
import Banner from "../../components/Banner";
import Dropdown from "../../components/Dropdown";

export default {
  name: 'HeaderComponent',
  
  components: { 
    Dropdown, 
    Banner, 
    SearchPopup 
  },

  directives: { 
    outsideClick 
  },

  mixins: [util],

  data() {
    return {
      headerSticky: false,
      topBannerLoaded: false,
      isTopBannerClosed: true,
      dropdown: false,
      searchPopup: false,
      searchFocused: false,
      searchedText: '',
      userCount: null,
      observer: null
    }
  },

  computed: {
    isXSmallerDevice() {
      return typeof window !== 'undefined' && window.innerWidth <= 576
    },
    headerLeft() {
      return this.headerLinks?.left || []
    },
    headerRight() {
      return this.headerLinks?.right || []
    },
    isPublic() {
      return parseInt(this.topBanner?.status) === this.status.PUBLIC
    },
    isLoggedIn() {
      return this.$auth?.loggedIn || false
    },
    cartCountCom() {
      return this.$auth?.user?.cart_count
    },
    username() {
      return this.$auth?.user?.name?.split(' ')[0]
    },
    email() {
      return this.setting?.email
    },
    phone() {
      return this.setting?.phone
    },
    sellerSignUp() {
      return this.setting?.vendor_registration
    },
    ...mapGetters('language', ['languages', 'currentLanguage']),
    ...mapGetters('common', ['site_setting', 'setting', 'topBanner', 'headerLinks']),
    ...mapGetters('listing', ['searched']),
    ...mapGetters('cart', ['cartCount'])
  },

  watch: {
    cartCountCom(value) {
      this.setCartCount(value)
    },
    '$route'() {
      this.setQFromRoute()
      this.closeDropdown()
    },
    searchedText() {
      if (!this.searchPopup && this.searchFocused) {
        this.emptySearchedSuggestion()
        this.openSearchPopup()
      }
    }
  },

  async mounted() {
    // Initialize data
    await this.fetchUserCount()
    this.setQFromRoute()
    this.updateSearch(this.searchedText)
    
    if (this.cartCountCom) {
      this.setCartCount(this.cartCountCom)
    }

    // Handle top banner
    this.$nextTick(() => {
      if (localStorage.getItem('topBannerClosed') !== null) {
        this.isTopBannerClosed = localStorage.getItem('topBannerClosed') === 'true'
        this.topBannerLoaded = true
      } else {
        this.isTopBannerClosed = false
        this.topBannerLoaded = true
      }
    })

    // Initialize intersection observer
    if (typeof window !== 'undefined' && this.$refs.headerWrapper) {
      const rootMargin = this.isXSmallerDevice ? '40px 0px 0px 0px' : '0px'
      
      this.observer = new IntersectionObserver(this.handleIntersection, {
        root: null,
        rootMargin,
        threshold: 0
      })

      this.observer.observe(this.$refs.headerWrapper)
    }
  },

  beforeDestroy() {
    if (this.observer) {
      this.observer.disconnect()
    }
  },

  methods: {
    async fetchUserCount() {
      try {
        const response = await this.$axios.get('/api/user/user-count')
        if (response.data?.status === 'success') {
          this.userCount = response.data.count
        }
      } catch (error) {
        console.error('Error fetching user count:', error)
      }
    },

    handleIntersection(entries) {
      entries.forEach((entry) => {
        this.headerSticky = !entry.isIntersecting
      })
    },

    async selectedLanguage(data) {
      document.cookie = `currentLanguage=${data.key}; path=/; max-age=${365 * 24 * 60 * 60}`
      location.reload()
    },

    topBannerClosed() {
      localStorage.setItem('topBannerClosed', 'true')
      this.isTopBannerClosed = true
    },

    openSearchPopup() {
      if (this.searchedText.length > 0) {
        this.searchPopup = true
      }
      this.searchFocused = true
    },

    blurSearchInput() {
      this.searchFocused = false
      this.closeSearchPopup()
    },

    closeSearchPopup() {
      setTimeout(() => {
        this.searchPopup = false
      }, 100)
    },

    setQFromRoute() {
      this.searchedText = this.$route?.query?.q || ''
    },

    search() {
      if (this.searchedText && (this.searchedText !== this.searched || this.$route.name !== 'search')) {
        this.$router.push({ path: `/search?q=${this.searchedText}` })
        this.updateSearch(this.searchedText)
      }
    },

    async loggingOut() {
      try {
        await this.$auth.logout()
        this.closeDropdown()
      } catch (error) {
        return this.$nuxt.error(error)
      }
    },

    closeDropdown() {
      this.dropdown = false
    },

    ...mapActions('language', ['setDefaultLanguage', 'getLangData']),
    ...mapActions('cart', ['emptyCartProduct', 'setCartCount']),
    ...mapActions('listing', ['updateSearch', 'emptySearchedSuggestion'])
  }
}
</script>