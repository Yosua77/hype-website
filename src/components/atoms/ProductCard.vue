<script setup>
defineProps({
  product: {
    type: Object,
    required: true,
  },
  brand: {
    type: String,
    required: true,
  },
  responsive: {
    type: Boolean,
    default: true,
  },
});
</script>

<script>
export default {
  computed: {
    title() {
      return this.product.title
        .toLowerCase()
        .replace(/[^\w\s]/gi, "")
        .replace(/ /g, "-");
    },
    subtitle() {
      return this.product.subtitle
        .toLowerCase()
        .replace(/[^\w\s]/gi, "")
        .replace(/ /g, "-");
    },
  },
};
</script>

<template>
  <div class="product-card" :class="responsive ? 'responsive' : ''">
    <RouterLink :to="`/brands/${brand}/${title}-${subtitle}`">
      <img
        class="product-image"
        v-lazy="{
          src: product.img[0],
          loading: '/images/products/default.jpg',
          error: '/images/products/default.jpg',
        }"
        alt="Product"
        loading="lazy" /><img
        class="product-logo"
        :class="brand"
        :src="`/logos/${brand}.svg`"
        alt="Logo" />
      <img
        class="product-image-alternate"
        v-lazy="{
          src: product.img[1],
          loading: '/images/products/default.jpg',
          error: '/images/products/default.jpg',
        }"
        alt="Product"
        loading="lazy" /><img
        class="product-logo"
        :class="brand"
        :src="`/logos/${brand}.svg`"
        alt="Logo"
    /></RouterLink>

    <div class="product-detail">
      <h5 class="product-title">{{ product.title }} {{ product.subtitle }}</h5>
      <p class="product-subtitle">
        <RouterLink :to="`/brands/${brand}`">{{ brand }}</RouterLink> /
        <RouterLink :to="`/search?q=${product.gender}`">{{
          product.gender
        }}</RouterLink>
        /
        <RouterLink :to="`/search?q=${product.type}`">{{
          product.type
        }}</RouterLink>
      </p>
      <!-- <h5 class="product-price">
        {{ formatRupiah(product.price + "", "IDR") }}
      </h5> -->
    </div>
  </div>
</template>

<style scoped>
.product-card {
  position: relative;
  color: var(--color-text-dark);
}
.product-card.fluid {
  width: 100%;
}

.product-card > a:hover .product-image {
  opacity: 0;
}

.product-card > a:hover .product-image-alternate {
  opacity: 1;
}

.product-image {
  transition: all 0.3s cubic-bezier(0.65, 0, 0.35, 1);
}

.product-image-alternate {
  position: absolute;
  left: 0;
  top: 0;
  opacity: 0;
  transition: all 0.3s cubic-bezier(0.65, 0, 0.35, 1);
}

.product-image,
.product-image-alternate {
  width: 100%;
  aspect-ratio: 4/5;
  object-fit: cover;
}

.product-title {
  font-weight: 600;
}

.product-subtitle > * {
  color: var(--color-text-dark);
}

.product-logo {
  height: 3rem;
  position: absolute;
  right: 1rem;
  top: 1rem;
}

.nike {
  height: 0.65rem !important;
}

.adidas {
  height: 2rem !important;
}

.product-logo {
  opacity: 0;
}

img[lazy="loaded"] + .product-logo {
  opacity: 1;
}

@media screen and (max-width: 560px) {
  .responsive .product-title {
    font-size: var(--fontSize-p);
  }

  .responsive .product-subtitle {
    text-transform: uppercase;
    font-weight: 600;
    opacity: 0.55;
    font-size: var(--fontSize-small);
    letter-spacing: 0.05rem;
  }

  .product-logo {
    height: 1.5rem;
  }

  .givenchy {
    height: 0.5rem;
  }
}
</style>

<!-- 6-18, 6-12, 3-18, ?-? -->
