<script setup>
document.title = "Product Detail - Hype";
defineProps({
  scrollPos: {
    type: Number,
    default: 0,
  },
  screenWidth: {
    type: Number,
    default: 0,
  },
});
</script>

<script>
import InputButton from "../components/atoms/InputButton.vue";
import InputText from "../components/atoms/InputText.vue";
import { Swiper, SwiperSlide } from "swiper/vue";
import "swiper/css";

export default {
  components: {
    Swiper,
    SwiperSlide,
  },
  data() {
    return {
      product: null,
      selectedSize: null,
      selectedColor: "",
      quantityValue: 1,
      brand: this.$route.params.brand,
      state: "default",
      carousel: null,
    };
  },
  created() {
    const data = JSON.parse(localStorage.getItem("brands"));
    for (let i = 0; i < data.length; i++) {
      for (let j = 0; j < data[i].products.length; j++) {
        let title = data[i].products[j].title.toLowerCase();
        title = title.replace(/[^\w\s]/gi, "").replace(/ /g, "-");
        let subtitle = data[i].products[j].subtitle.toLowerCase();
        subtitle = subtitle.replace(/[^\w\s]/gi, "").replace(/ /g, "-");
        if (`${title}-${subtitle}` == this.$route.params.name) {
          this.product = data[i].products[j];
        }
      }
    }
  },
  mounted() {
    this.carousel = document.querySelector(".carousel-image").swiper;
  },
  methods: {
    formatRupiah(angka, prefix) {
      var number_string = angka.replace(/[^,\d]/g, "").toString(),
        split = number_string.split(","),
        sisa = split[0].length % 3,
        rupiah = split[0].substr(0, sisa),
        ribuan = split[0].substr(sisa).match(/\d{3}/gi);

      if (ribuan) {
        const separator = sisa ? "," : "";
        rupiah += separator + ribuan.join(",");
      }

      rupiah = split[1] != undefined ? rupiah + "," + split[1] : rupiah;
      return prefix == undefined ? rupiah : rupiah ? "IDR " + rupiah : "";
    },
    addCart() {
      if (this.selectedSize == null) {
        this.state = "error";
      } else {
        let cart = JSON.parse(localStorage.getItem("cart")) || [];
        const foundIndex = cart.findIndex(
          (item) =>
            item.title + " " + item.subtitle ==
              this.product.title + " " + this.product.subtitle &&
            item.selectedSize == this.selectedSize
        );
        if (foundIndex != -1) {
          cart[foundIndex].qty = parseInt(cart[foundIndex].qty);
          cart[foundIndex].qty += parseInt(this.quantityValue * 1);
        } else {
          this.product.selectedSize = this.selectedSize;
          this.product.qty = this.quantityValue;
          this.product.brand = this.brand;
          cart.push(this.product);
        }
        localStorage.setItem("cart", JSON.stringify(cart));
        this.$emit("cartChanged");
      }
    },
    selectSize(size) {
      this.selectedSize = size;
      this.state = "default";
    },
    slideChange(direction) {
      if (direction == "right") {
        this.carousel.slideNext();
      } else if (direction == "left") {
        this.carousel.slidePrev();
      }
    },
  },
};
</script>

<template>
  <div class="container-detail">
    <div class="img-detail">
      <img
        v-for="image in product.img"
        :key="image"
        class="product-image"
        v-lazy="{
          src: image,
          loading: '/images/products/default.jpg',
          error: '/images/products/default.jpg',
        }"
        alt="Product"
        loading="lazy"
      />
    </div>
    <div class="product-detail">
      <div class="product-detail-wrapper">
        <div class="detail-header">
          <p class="product-subtitle">
            <RouterLink :to="`/brands/${brand}`">{{ brand }}</RouterLink> /
            <RouterLink :to="`/brands/${brand}?gender=${product.gender}`">{{
              product.gender
            }}</RouterLink>
            /
            <RouterLink :to="`/brands/${brand}?type=${product.type}`">{{
              product.type
            }}</RouterLink>
          </p>
          <h4 class="product-title">
            {{ product.title }} {{ product.subtitle }}
          </h4>
          <h5 class="product-price">
            {{ formatRupiah(product.price + "", "IDR") }}
          </h5>
        </div>
        <div class="img-detail-mobile">
          <swiper class="carousel-image">
            <swiper-slide
              v-for="image in product.img"
              :key="image"
              class="carousel-image-slide"
              ><img
                class="carousel-image-content"
                v-lazy="{
                  src: image,
                  loading: '/images/products/default.jpg',
                  error: '/images/products/default.jpg',
                }"
                alt="Product"
                loading="lazy"
            /></swiper-slide>
          </swiper>
          <div
            class="carousel-control carousel-prev"
            v-if="carousel != null"
            :class="carousel.isBeginning ? '' : 'active'"
            @click="slideChange('left')"
          >
            <svg
              height="10"
              viewBox="0 0 50 16"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                d="M0.292893 7.2929C-0.0976311 7.68342 -0.097631 8.31659 0.292893 8.70711L6.65685 15.0711C7.04738 15.4616 7.68054 15.4616 8.07107 15.0711C8.46159 14.6805 8.46159 14.0474 8.07107 13.6569L2.41421 8L8.07107 2.34315C8.46159 1.95263 8.46159 1.31946 8.07107 0.928936C7.68054 0.538412 7.04738 0.538412 6.65685 0.928936L0.292893 7.2929ZM49.5 7L1 7L1 9L49.5 9L49.5 7Z"
                fill="currentColor"
              />
            </svg>
          </div>
          <div
            class="carousel-control carousel-next"
            v-if="carousel != null"
            :class="carousel.isEnd ? '' : 'active'"
            @click="slideChange('right')"
          >
            <svg
              height="10"
              viewBox="0 0 50 16"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                d="M49.2071 8.70711C49.5976 8.31658 49.5976 7.68342 49.2071 7.29289L42.8431 0.928932C42.4526 0.538408 41.8195 0.538408 41.4289 0.928932C41.0384 1.31946 41.0384 1.95262 41.4289 2.34315L47.0858 8L41.4289 13.6569C41.0384 14.0474 41.0384 14.6805 41.4289 15.0711C41.8195 15.4616 42.4526 15.4616 42.8431 15.0711L49.2071 8.70711ZM0 9H48.5V7H0V9Z"
                fill="currentColor"
              />
            </svg>
          </div>
          <div class="carousel-navigation" v-if="carousel != null">
            <div
              class="carousel-navigation-item"
              v-for="n in carousel.slides.length"
              :key="n"
              :class="carousel.activeIndex + 1 == n ? 'active' : ''"
            ></div>
          </div>
        </div>
        <div class="size-section">
          <h5 class="section-header">Size</h5>
          <div class="product-sizes">
            <div
              v-for="size in product.size"
              :key="size"
              class="size-variant"
              @click="selectSize(size)"
              :class="selectedSize == size ? 'active' : ''"
            >
              {{ size }}
            </div>
          </div>
          <p class="error-text" v-show="state == 'error'">Please select size</p>
        </div>
        <div class="qty-section">
          <h5 class="section-header">Quantity</h5>
          <div class="qty-counter">
            <InputText
              :withLabel="false"
              name="qtyInput"
              type="quantity"
              v-model="quantityValue"
            />
          </div>
          <div class="button-add">
            <div class="subtotal">
              <p style="font-weight: 400">Subtotal</p>
              <h5 style="font-weight: bold">
                {{ formatRupiah(quantityValue * product.price + "", "IDR") }}
              </h5>
            </div>
            <InputButton :text="`ADD TO CART`" @click="addCart()" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container-detail {
  width: 100%;
  display: flex;
  gap: 2rem;
  margin-block: 8rem;
  padding-inline: 3rem;
}

.img-detail {
  flex: 1;
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-gap: 1rem;
}

.featured-carousel {
  width: 100%;
  padding-inline: 3rem;
}

.carousel-item {
  width: 100%;
  transition: all 0.3s cubic-bezier(0.65, 0, 0.35, 1);
}

.carousel-item:not(.swiper-slide-active) {
  opacity: 0;
  filter: brightness(55%);
}

.product-detail {
  position: relative;
}

.subtotal {
  width: 100%;
  display: flex;
  justify-content: space-between;
}

.product-detail-wrapper {
  width: 24rem;
  display: flex;
  flex-flow: column;
  gap: 2rem;
  position: sticky;
  top: 8rem;
}

.product-title {
  font-weight: 600;
}

.product-subtitle > * {
  color: var(--color-text-dark);
}

.product-image {
  width: 100%;
  aspect-ratio: 4/5;
  object-fit: cover;
  border: solid 2px var(--clr-neutral-500);
}

.carousel-image {
  display: none;
}

.size-section,
.color-section,
.qty-section {
  width: 100%;
  display: flex;
  flex-flow: column;
  gap: 1rem;
}

.product-sizes,
.product-colors {
  width: 100%;
  --auto-grid-min-size: 3rem;
  display: grid;
  grid-template-columns: repeat(
    auto-fill,
    minmax(var(--auto-grid-min-size), 1fr)
  );
  grid-gap: 0.5rem;
}

.size-variant {
  width: 3rem;
  display: grid;
  place-items: center;
  aspect-ratio: 1/1;
  text-align: center;
  border: solid 2px var(--clr-neutral-300);
  cursor: pointer;
  font-weight: 600;
  transition: all 0.1s cubic-bezier(0.65, 0, 0.35, 1);
}

.size-variant:hover {
  border: solid 2px var(--clr-neutral-800);
}
.size-variant.active {
  color: var(--color-text-light);
  background-color: var(--color-container-dark);
  border-color: var(--color-container-dark);
}

.container-color {
  width: calc(100% - 3rem);
  display: inline-flex;
  gap: 1rem;
  cursor: pointer;
}

.color-variant {
  width: 3rem;
  aspect-ratio: 1/1;
  display: grid;
  place-items: center;
  border: solid 2px var(--color-container-dark);
  cursor: pointer;
  position: relative;
}

.color-variant::after {
  width: 5px;
  height: 5px;
  background-color: var(--color-container-dark);
  content: "";
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  filter: invert() grayscale(200%);
  mix-blend-mode: difference;
  opacity: 0;
  transition: all 0.1s cubic-bezier(0.65, 0, 0.35, 1);
}

.color-variant:hover::after {
  opacity: 1;
}

.qty-counter {
  display: flex;
  flex-direction: row;
  gap: 1.5rem;
  height: 5vh;
  margin-bottom: 2rem;
}

.button-add {
  width: 100%;
  display: flex;
  flex-flow: column;
  gap: 1rem;
}

.carousel-control {
  display: none;
}

.error-text {
  color: var(--clr-error-500);
}

@media screen and (max-width: 1600px) {
  .swiper-slide-next + .carousel-item {
    opacity: 0.3;
    filter: brightness(55%);
  }
}

@media screen and (max-width: 1300px) {
  .img-detail {
    grid-template-columns: 1fr;
  }
}

@media screen and (max-width: 992px) {
  .container-detail {
    flex-flow: column;
  }

  .product-detail-wrapper {
    width: 100%;
  }

  .carousel-image {
    width: 100%;
    height: 100%;
    display: flex;
  }

  .img-detail-mobile {
    position: relative;
    overflow-x: hidden;
  }

  .carousel-image-content {
    width: 100%;
    height: 100%;
    aspect-ratio: 4/5;
    object-fit: cover;
    border: solid 2px var(--clr-neutral-300);
  }

  .container-detail {
    padding-inline: 1rem;
    margin-top: 6rem;
  }

  .img-detail {
    display: none;
  }

  .carousel-control {
    position: absolute;
    width: 20%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    /* background-color: var(--color-container-dark); */
    color: var(--color-text-dark);
    top: 0;
    z-index: 10;
    cursor: pointer;
  }

  .carousel-control:not(.active) {
    pointer-events: none;
    opacity: 0.5;
  }

  .carousel-prev {
    left: 0;
  }

  .carousel-next {
    right: 0;
  }
  .button-add {
    width: 100%;
  }

  .size-section,
  .color-section,
  .qty-section {
    width: 100%;
  }

  .carousel-navigation {
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 0.5rem;
    position: absolute;
    bottom: 2rem;
    z-index: 10;
  }

  .carousel-navigation-item {
    width: 3rem;
    height: 5px;
    background-color: var(--color-container-dark);
    border: solid 1px var(--color-background);
    opacity: 0.3;
    transition: all 0.3s cubic-bezier(0.65, 0, 0.35, 1);
  }

  .carousel-navigation-item.active {
    opacity: 1;
  }
}

@media screen and (max-width: 768px) {
  .container-detail {
    padding-inline: 1rem;
    margin-top: 2rem;
  }
}
</style>
