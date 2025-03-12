<template>
  <div class="artwork-box">
    <div class="artworks blur" v-if="!minimizedCards.has(index)">
      <div
          v-for="(item, index) in spaceItems"
          :key="item.id"
          class="artwork-card"
          v-show="index === currentSlide"
          :class="{ minimized: minimizedCards.has(index) }"
      >
        <div class="card-content" v-if="!minimizedCards.has(index)">
          <div class="artwork-info">
            <div class="artwork-name">{{ item.product.name }}</div>
              <a :href="item.product.shop_url" target="_blank">
                  <div class="artist-name">
                      {{ item.product.s_firstname }} {{ item.product.s_lastname }}
                  </div>
              </a>
            <div class="price" v-html="item.product.regular_price"></div>
          </div>

          <div class="card-actions">
            <template v-if="!isPurchaseClient && isMultipleProject">
              <button
                  class="action primary"
                  :class="{ select: !item.is_approved, selected: item.is_approved }"
                  @click="approveArt(item)"
              >
                {{ item.is_approved ? 'Cancel' : 'Approve' }}
              </button>
            </template>
            <template v-if="isPurchaseClient || !isMultipleProject">
              <!-- Future Add To Cart Button -->
            </template>
            <button class="artwork-screen" @click="openPopupArtView(item)">
              <span class="btn-view-artwork">View Artwork</span>
            </button>
          </div>
        </div>
      </div>
        <button class="minimize-button" @click="minimizeCard(index)"></button>
    </div>
      <button class="maximize-button blur" v-if="minimizedCards.has(index)" @click="minimizeCard(index)"></button>
    <div class="carousel-controls blur">
      <button @click="prevSlide" :disabled="currentSlide === 0">‹</button>
      <span>{{ currentSlide + 1}}/{{ spaceItems.length }}</span>
      <button @click="nextSlide" :disabled="currentSlide === spaceItems.length - 1">›</button>
    </div>

  </div>
</template>

<script>
import { ref, onMounted, watch } from 'vue';

export default {
  props: {
    spaceItems: Array,
    isPurchaseClient: Boolean,
    isMultipleProject: Boolean,
    currentSpaceItem: Object
  },
  setup(props, {emit}) {
    const currentSlide = ref(0);
    const minimizedCards = ref(new Set());

    const approveArt = (item) => {
      item.is_approved = !item.is_approved;
    };

    const openPopupArtView = (item) => {
      console.log('Opening artwork popup', item);
    };

    const changeSlide = (direction) => {
      currentSlide.value = Math.min(
          Math.max(currentSlide.value + direction, 0),
          props.spaceItems.length - 1
      );
    };

    const minimizeCard = (index) => {
      if (minimizedCards.value.has(index)) {
        minimizedCards.value.delete(index);
      } else {
        minimizedCards.value.add(index);
      }
    };

    watch(currentSlide, (newSlideIndex) => {
      emit('update:currentSpaceItem', props.spaceItems[newSlideIndex]);
    });

    onMounted(() => {
      const params = new URLSearchParams(window.location.search);
      const productId = params.get('product');
      if (productId) {
        const index = props.spaceItems.findIndex(item => item.product.id == productId);
        if (index !== -1) currentSlide.value = index;
      }
    });

    return {
      currentSlide,
      approveArt,
      openPopupArtView,
      prevSlide: () => changeSlide(-1),
      nextSlide: () => changeSlide(1),
      minimizeCard,
      minimizedCards
    };
  },
};
</script>
