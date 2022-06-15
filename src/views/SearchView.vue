<script setup>
import SearchMagnifier from "../components/icons/SearchMagnifier.vue";
import ProductCard from "../components/atoms/ProductCard.vue";

document.title = "Hype - Brands";

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
export default {
  components: { SearchMagnifier },
  data() {
    return {
      dataProducts: null,
      inputVal: "",
      inputValAC: "",
      dataProductAC: [],
      showedProducts: [],
      object: null,
      loading: false,
      debounce: 2,
      debounceInterval: false,
      autofills: [],
      inputFocus: false,
    };
  },
  created() {
    this.dataProducts = JSON.parse(localStorage.getItem("brands"));
    this.initialValue();
    if (this.$route.query.q) {
      this.inputVal = this.$route.query.q || "";
      this.populateShowedProducts();
    }
  },
  mounted() {
    window.scrollTo(0, 0);
  },
  methods: {
    initialValue() {
      for (let i = 0; i < this.dataProducts.length; i++) {
        for (let j = 0; j < this.dataProducts[i].products.length; j++) {
          let dataProd = {
            name: this.dataProducts[i].products[j].title.concat(
              " ",
              this.dataProducts[i].products[j].subtitle
            ),
          };
          this.dataProductAC.push(dataProd);
        }
      }
    },
    keyUpHandle(e) {
      console.log(e.keyCode);
      this.showedProducts = [];
      clearInterval(this.debounceInterval);
      this.debounceInterval = false;
      if (e.keyCode == 40 && this.inputValAC.length != 0) {
        this.inputVal = this.inputValAC;
        this.inputValAC = "";
        this.autofills = [];
      }
      if (!this.inputValAC.startsWith(this.inputVal)) {
        this.inputValAC = "";
      }
      if (e.keyCode != 13) {
        this.debounce = 1;
        if (this.inputVal.length == 0) {
          this.inputValAC = "";
          this.showedProducts = [];
          this.autofills = [];
        }
        if (
          this.debounceInterval == false &&
          (/[a-zA-Z0-9-_ ]/.test(String.fromCharCode(e.keyCode)) ||
            e.keyCode == 8 ||
            e.keyCode == 40)
        ) {
          this.debounceInterval = setInterval(() => {
            this.debounce--;
            console.log(this.debounce);
            if (this.debounce == 0) {
              if (e.keyCode != 40) {
                if (this.screenWidth <= 768 && this.inputVal.length > 0)
                  this.populateAutofill();
                else this.autoCompleteSearch(e);
              }
              clearInterval(this.debounceInterval);
              this.debounceInterval = false;
            }
          }, 500);
        }
      } else {
        this.loadingData();
      }
    },
    loadingData() {
      clearInterval(this.debounceInterval);
      this.debounceInterval = false;
      this.debounce = 2;
      this.loading = true;
      this.debounceInterval = setInterval(() => {
        this.debounce--;
        if (this.debounce == 0) {
          this.populateShowedProducts();
          this.loading = false;
        }
      }, 1000);
    },
    autoCompleteSearch() {
      let val = this.inputVal;
      let object = "";
      if (val.length < 1) {
        this.inputValAC = "";
        this.showedProducts = [];
        this.$router.replace({
          name: "SearchProduct",
        });
        return;
      }
      object = this.findValueByPrefix(val);
      if (
        object !== null &&
        (this.inputVal.length < this.inputValAC.length || this.inputValAC == "")
      ) {
        this.inputValAC = object.name.toLowerCase();
      } else {
        this.inputValAC = "";
      }
    },
    populateAutofill() {
      this.autofills = [];
      for (let i = 0; i < this.dataProducts.length; i++) {
        const products = this.dataProducts[i].products;
        for (let j = 0; j < products.length; j++) {
          const product = products[j];
          const fullName = `${product.title} ${product.subtitle}`;

          if (fullName.toLowerCase().includes(this.inputVal.toLowerCase())) {
            this.autofills.push(fullName);
          }
        }
      }
    },
    findValueByPrefix(prefix) {
      for (let i = 0; i < this.dataProductAC.length; i++) {
        let object = this.dataProductAC[i];
        if (object.name.toLowerCase().startsWith(prefix.toLowerCase())) {
          return object;
        }
      }
      return null;
    },
    populateShowedProducts() {
      this.showedProducts = this.findBrandBySearchValue();
      if (this.showedProducts.length == 0) {
        this.showedProducts = false;
      }
      this.$router.replace({
        name: "SearchProduct",
        query: { q: this.inputVal },
      });
      // Brand name: item.name
      // Product: i
    },
    findBrandBySearchValue() {
      const index = this.dataProducts.findIndex((item) => {
        return item.name.toLowerCase() == this.inputVal.toLowerCase();
      });
      console.log(index);
      if (index == -1) {
        return this.findTypesBySearchValue();
      }
      return [this.dataProducts[index]];
    },
    findTypesBySearchValue() {
      let products = [];
      for (let i = 0; i < this.dataProducts.length; i++) {
        const brand = this.dataProducts[i];
        // Iterate products
        for (let j = 0; j < brand.products.length; j++) {
          if (
            brand.products[j].type.toLowerCase() == this.inputVal.toLowerCase()
          ) {
            //Jika value input terdapat di typenya product,

            //cek products apakah brand product sudah ada
            const brandIndex = products.findIndex((item) => {
              return item.name == brand.name;
            });

            if (brandIndex == -1) {
              //Jika tidak ada brand product didalam products
              products.push({
                name: brand.name,
                products: [brand.products[j]],
              });
            } else {
              //Jika brand product sudah ada didalam products
              products[brandIndex].products.push(brand.products[j]);
            }
          }
        }
      }

      if (products.length > 0) {
        return products;
      }

      return this.findGendersBySearchValue();
    },
    findGendersBySearchValue() {
      let products = [];
      for (let i = 0; i < this.dataProducts.length; i++) {
        const brand = this.dataProducts[i];
        for (let j = 0; j < brand.products.length; j++) {
          if (
            brand.products[j].gender.toLowerCase() ==
            this.inputVal.toLowerCase()
          ) {
            //Jika value input terdapat di gendernya product,

            //cek products apakah brand product sudah ada
            const brandIndex = products.findIndex((item) => {
              return item.name == brand.name;
            });

            if (brandIndex == -1) {
              //Jika tidak ada brand product didalam products
              products.push({
                name: brand.name,
                products: [brand.products[j]],
              });
            } else {
              //Jika brand product sudah ada didalam products
              products[brandIndex].products.push(brand.products[j]);
            }
          }
        }
      }
      if (products.length > 0) {
        return products;
      }

      return this.findProductsBySearchValue();
    },
    findProductsBySearchValue() {
      const products = [];

      for (let i = 0; i < this.dataProducts.length; i++) {
        const brand = this.dataProducts[i];

        // Iterate products
        for (let j = 0; j < brand.products.length; j++) {
          // Concat title dan subtitle
          const fullName =
            brand.products[j].title + " " + brand.products[j].subtitle;

          if (fullName.toLowerCase().includes(this.inputVal.toLowerCase())) {
            //Jika value input terdapat di fullName,

            //cek products apakah brand product sudah ada
            const brandIndex = products.findIndex((item) => {
              return item.name == brand.name;
            });

            if (brandIndex == -1) {
              //Jika tidak ada brand product didalam products
              products.push({
                name: brand.name,
                products: [brand.products[j]],
              });
            } else {
              //Jika brand product sudah ada didalam products
              products[brandIndex].products.push(brand.products[j]);
            }
          }
        }
      }
      return products;
    },
    autofillHandler(value) {
      this.inputVal = value;
      this.autofills = [];
      this.populateShowedProducts();
    },
  },
};
</script>

<template>
  <div class="container-search">
    <div class="search-input-section">
      <div
        class="input-wrapper"
        :class="loading && inputVal != 0 ? 'loading' : ''"
      >
        <p class="ac-helper" :class="inputValAC.length > 0 ? 'showing' : ''">
          Press ↓ to autocomplete
        </p>
        <input
          type="text"
          class="input-text-search"
          placeholder="Type your search here"
          v-model="inputVal"
          @keyup="keyUpHandle"
          @focusin="inputFocus = true"
          @focusout="inputFocus = false"
          :class="inputFocus ? 'input-focus' : ''"
        />
        <input
          type="text"
          class="input-text-search ac"
          v-model="inputValAC"
          disabled
        />
        <SearchMagnifier class="searchIcon" @click="loadingData" />
        <div
          class="autofill-mobile"
          :class="autofills.length > 0 && inputFocus ? 'showing' : ''"
        >
          <p
            v-for="title in autofills"
            :key="title"
            @click="autofillHandler(title)"
            tabindex="0"
          >
            {{ title }}
          </p>
        </div>
      </div>
    </div>
    <h5 v-show="typeof showedProducts != 'object' && showedProducts == false">
      Sorry, we could not find anything for "{{ inputVal }}".
    </h5>
    <div
      class="search-product-wrapper"
      v-show="inputVal.length > 0 || showedProducts.length > 0"
    >
      <div
        class="search-product-section"
        v-for="brand in showedProducts"
        :key="brand.name"
      >
        <div class="info-search">
          <h5 class="show-info-brand">{{ brand.name }}</h5>
          <p class="show-info">Found {{ brand.products.length }} Products</p>
        </div>
        <div class="grid-catalogue">
          <ProductCard
            v-for="product in brand.products"
            :key="`${product.title}_${product.subtitle}`"
            :product="product"
            :brand="brand.name"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container-search {
  width: 100%;
  min-height: calc(100vh - 12rem);
  padding-top: calc(72px + 4rem);
  display: flex;
  flex-flow: column;
  align-items: center;
  gap: 4rem;
}

.search-input-section {
  width: 100%;
  padding-inline: 3rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.ac-helper {
  position: absolute;
  left: 0;
  top: 0;
  z-index: -1;
  transition: all 0.3s cubic-bezier(0.65, 0, 0.35, 1);
}

.ac-helper.showing {
  top: -2rem;
}

.input-wrapper {
  flex: 1;
  position: relative;
  height: max-content;
}

.input-text-search {
  width: 100%;
  position: absolute;
  font-size: var(--fontSize-h3);
  background: none;
  outline: none;
  border: none;
  z-index: 7;
}

.input-text-search.ac {
  position: static;
  background-color: var(--color-background);
  color: var(--clr-neutral-600);
  z-index: 1;
}

.searchIcon {
  height: 100%;
  right: 0;
  position: absolute;
  cursor: pointer;
  z-index: 8;
  padding: 0.75rem 0.875rem;
  transition: all 0.1s cubic-bezier(0.65, 0, 0.35, 1);
}

.searchIcon:hover {
  background-color: var(--color-container-dark);
  color: var(--color-text-light);
}

.search-product-wrapper {
  width: 100%;
  display: flex;
  flex-flow: column;
  gap: 2rem;
}

.input-wrapper::after {
  content: "";
  display: block;
  width: 0;
  height: 2px;
  background: var(--color-container-dark);
  transition: width 0s cubic-bezier(0.65, 0, 0.35, 1);
}

.input-wrapper.loading::after {
  width: 100%;
  transition: width 2s cubic-bezier(0.65, 0, 0.35, 1);
}

.search-product-section {
  width: 100%;
  padding-inline: 3rem;
  display: flex;
  gap: 1rem;
  justify-content: flex-start;
  align-items: flex-start;
  flex-flow: column;
}

.info-search {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-flow: row;
}

.show-info-brand {
  flex: 1;
  text-transform: uppercase;
  letter-spacing: 0.05rem;
}

.show-info {
  font-weight: bold;
}

.grid-catalogue {
  width: 100%;
  --auto-grid-min-size: 24rem;
  display: grid;
  grid-template-columns: repeat(
    auto-fill,
    minmax(var(--auto-grid-min-size), 1fr)
  );
  margin-bottom: 1rem;
  grid-gap: 1rem;
}

.autofill-mobile {
  display: flex;
  flex-flow: column;
  height: 0;
  overflow-y: auto;
  visibility: hidden;
  border: solid 2px var(--color-container-dark);
  border-top: none;
  transition: all 0.3s cubic-bezier(0.65, 0, 0.35, 1);
}

.autofill-mobile.showing {
  visibility: visible;
  height: 12rem;
}

.autofill-mobile * {
  font-weight: 600;
  cursor: pointer;
  padding: 1rem 0.875rem;
}

.autofill-mobile *:focus {
  background-color: var(--clr-neutral-500);
}

@media screen and (max-width: 992px) {
  .search-input-section,
  .search-product-section {
    padding-inline: 1rem;
  }
}

@media screen and (max-width: 768px) {
  .search-input-section {
    padding-inline: 1rem;
  }

  .container-search {
    padding-top: 2rem;
  }

  .input-text-search {
    height: 56px;
    font-size: var(--fontSize-h5);
    border: solid 2px var(--clr-neutral-300);
    padding: 0.75rem 0.875rem;
    transition: border 0.3s cubic-bezier(0.65, 0, 0.35, 1);
  }

  .input-text-search.input-focus {
    border: solid 2px var(--color-container-dark);
  }

  .ac-helper {
    display: none;
  }

  .input-text-search.ac {
    display: none;
  }

  .input-text-search {
    position: static;
  }

  .searchIcon {
    height: 56px;
    padding: 0.75rem 0.875rem;
    border: none;
    /* background-color: var(--color-container-dark); */
    /* color: var(--color-text-light); */
  }

  .grid-catalogue {
    grid-template-columns: 1fr 1fr;
    row-gap: 2rem;
  }
}
</style>
