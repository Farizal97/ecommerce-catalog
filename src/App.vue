<template>
  <div :class="['page', pageClass]">
    <div class="design-background">
      <header class="page-label">
        <span v-if="isWomen">Page - Women section</span>
        <span v-else-if="isMen">Page - Men section</span>
        <span v-else>Page - Unavailable product</span>
      </header>

      <main class="product-card">
        <template v-if="!loading && !error && product">
          <div class="product-image-area">
            <img :src="product.image" :alt="product.title" class="product-image">
          </div>

          <div class="product-content">
            <h1 class="product-title">{{ product.title }}</h1>
            <div class="category-line">{{ product.category }}</div>

            <div class="rating-line">
              <span>{{ product.rating ? product.rating.rate.toFixed(1) : "0.0" }}/5</span>
              <span class="rating-dots">
                <i v-for="n in 5" :key="n" :class="{ active: n <= roundedRating }"></i>
              </span>
            </div>

            <p class="description">{{ product.description }}</p>

            <div class="divider"></div>

            <div class="bottom-row">
              <div class="price">${{ formatPrice(product.price) }}</div>
              <div class="actions">
                <button class="buy-button" @click="buyProduct">Buy now</button>
                <button class="next-button" @click="nextProduct">Next product</button>
              </div>
            </div>
          </div>
        </template>

        <template v-else-if="!loading && !error && isUnavailable">
          <div class="unavailable-content">
            <div class="sad-face">
              <span class="eye eye-left"></span>
              <span class="eye eye-right"></span>
              <span class="sad-mouth"></span>
            </div>
            <p class="unavailable-text">This product is unavailable to show</p>
            <button class="unavailable-button" @click="nextProduct">Next product</button>
          </div>
        </template>

        <template v-else-if="loading">
          <div class="loading-state">
            <div class="spinner"></div>
            <p>Loading product...</p>
          </div>
        </template>

        <template v-else>
          <div class="loading-state">
            <p>{{ error }}</p>
            <button class="next-button error-button" @click="fetchProduct">Try again</button>
          </div>
        </template>
      </main>
    </div>
  </div>
</template>

<script>
const API_URL = "https://fakestoreapi.com/products";

export default {
  name: "App",

  data() {
    return {
      index: 1,
      product: null,
      loading: false,
      error: null,
      unavailable: false
    };
  },

  computed: {
    isMen() {
      return this.product && this.product.category === "men's clothing";
    },

    isWomen() {
      return this.product && this.product.category === "women's clothing";
    },

    isUnavailable() {
      return this.unavailable;
    },

    pageClass() {
      if (this.isWomen) return "page-women";
      if (this.isMen) return "page-men";
      return "page-unavailable";
    },

    roundedRating() {
      if (!this.product || !this.product.rating) return 0;
      return Math.round(this.product.rating.rate);
    }
  },

  mounted() {
    this.fetchProduct();
  },

  methods: {
    async fetchProduct() {
      this.loading = true;
      this.error = null;
      this.product = null;
      this.unavailable = false;

      try {
        const response = await fetch(`${API_URL}/${this.index}`);

        if (!response.ok) {
          throw new Error("Gagal mengambil data produk dari API.");
        }

        const data = await response.json();

        // Hanya men's clothing dan women's clothing yang disimpan.
        if (
          data.category === "men's clothing" ||
          data.category === "women's clothing"
        ) {
          this.product = data;
        } else {
          this.product = null;
          this.unavailable = true;
        }
      } catch (err) {
        this.error = err.message || "Terjadi kesalahan saat menghubungi API.";
      } finally {
        this.loading = false;
      }
    },

    nextProduct() {
      // Increment index setiap tombol Next Product ditekan.
      this.index += 1;

      // API hanya menyediakan produk 1-20.
      if (this.index > 20) {
        this.index = 1;
      }

      this.fetchProduct();
    },

    buyProduct() {
      alert("Demo: fitur Buy now belum terhubung ke checkout.");
    },

    formatPrice(price) {
      return Number(price).toFixed(2);
    }
  }
};
</script>