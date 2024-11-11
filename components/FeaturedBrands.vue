<template>
  <div class="area home-section">
    <template v-if="brands && brands.length">
      <div class="brand-slider">
        <image-slider
          :image-count="brands.length"
          :per-view="sliderOptions.perView"
          :gap="30"
          :responsive="sliderOptions.responsive"
        >
          <template v-slot:content>
            <li v-for="(brand, index) in brands" :key="index" class="brand-tile">
              <brand-tile :brand="brand" />
            </li>
          </template>
        </image-slider>
      </div>
    </template>
  </div>
</template>

<script>
import ImageSlider from '~/components/ImageSlider';
import BrandTile from './BrandTile';

export default {
  name: 'FeaturedBrands',
  props: {
    title: {
      type: String,
      default: ''
    },
    itemList: {
      type: Array,
      default() {
        return [];
      }
    },
    hasFeaturedBanner: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      brands: this.itemList
    };
  },
  components: {
    ImageSlider,
    BrandTile
  },
  computed: {
    sliderOptions() {
      return {
        perView: 6,
        responsive: [6, 5, 4, 3, 2]
      };
    }
  }
};
</script>

