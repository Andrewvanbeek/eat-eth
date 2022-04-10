<template>
  <div class="q-pa-md" style="max-width: 100em">
    <q-banner class="bg-primary text-white">
      <h3>Checkout</h3>
      <template v-slot:action>
        Powered by https://world.openfoodfacts.org/ and
        https://.openfoodfacts.org
        <q-btn @click="clearCart()">Clear Cart</q-btn>
      </template>
    </q-banner>
    <div class="row" style="overflow: auto; min-height: 50em; max-height: 50em">
      <q-list bordered padding>
        <q-item v-for="(checkoutItem, index) in cart" v-bind:key="index">
          <q-item-section top thumbnail class="q-ml-none">
            <img :src="checkoutItem.imageUrl" />
          </q-item-section>

          <q-item-section>
            <q-item-label>{{ checkoutItem.name }}</q-item-label>
            <q-item-label caption>Price 0.01 eth</q-item-label>
          </q-item-section>

          <q-item-section side top>
            <q-item-label caption>Price 0.01 eth</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
    </div>
    <div>
      <q-btn
        color="black"
        class="full-width"
        @click="dialog = true"
        label="Checkout"
      />
    </div>

    <q-dialog
      v-model="dialog"
      persistent
      :maximized="true"
      transition-show="slide-up"
      transition-hide="slide-down"
    >
      <q-card class="bg-primary text-white">
        <q-bar>
          <q-space />

          <q-btn
            dense
            flat
            icon="minimize"
            @click="maximizedToggle = false"
            :disable="!maximizedToggle"
          >
            <q-tooltip v-if="maximizedToggle" class="bg-white text-primary"
              >Minimize</q-tooltip
            >
          </q-btn>
          <q-btn
            dense
            flat
            icon="crop_square"
            @click="maximizedToggle = true"
            :disable="maximizedToggle"
          >
            <q-tooltip v-if="!maximizedToggle" class="bg-white text-primary"
              >Maximize</q-tooltip
            >
          </q-btn>
          <q-btn dense flat icon="close" v-close-popup>
            <q-tooltip class="bg-white text-primary">Close</q-tooltip>
          </q-btn>
        </q-bar>
        <video-auth v-if="cart.length > 0" :groceryList="cart" />
      </q-card>
    </q-dialog>
  </div>
</template>

<script>
import VideoAuth from "../components/VideoAuth.vue";
export default {
  components: {
    VideoAuth,
  },
  async mounted() {
    var requestOptions = {
      method: "GET",
      redirect: "follow",
    };
    console.log(this.$router.currentRoute);
    const product = this.$router.currentRoute?.value?.params?.category;
    const catTitle = product.split(":")[1];

    const response = await fetch(
      `https://us.openfoodfacts.org/category/${catTitle}.json`,
      requestOptions
    );
    const data = await response.json();
    this.products = data.products.slice(0, 100);
  },
  data: () => ({
    products: null,
    newCartItem: null,
    dialog: false,
    maximizedToggle: true,
  }),
  computed: {
    cart() {
      const component = this;
      const newCartItem = this.newCartItem;
      try {
        const cart = localStorage.getItem("cart");
        const cartItems = JSON.parse(cart);
        return cartItems || [];
      } catch (e) {
        return [];
      }
    },
  },
  setup() {
    return {
      lorem:
        "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.",
    };
  },
  methods: {
    async goToproduct(product) {
      this.$router.replace(`/products/${product.id}`);
      // this.$router
    },
    async addToCart(product) {
      const cart = this.cart;
      console.log("cart", cart);
      const basicProduct = { name: product.product_name };
      cart.push(basicProduct);
      this.newCartItem = basicProduct;
      console.log("this is the cart now", cart);
      if (
        cart.find(function (item) {
          return item.name == this.newCartItem.name;
        }) == undefined
      )
        localStorage.setItem("cart", JSON.stringify(cart));
    },
    async goToCheckout() {
      this.$router.replace(`/checkout`);
      // this.$router
    },
    async clearCart() {
      await localStorage.removeItem("cart")
      window.location.reload()
    }
  },
};
</script>