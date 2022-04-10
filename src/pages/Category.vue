<template>
  <div>
   <q-banner class="bg-primary text-white">
            Powered by https://world.openfoodfacts.org/ and https://.openfoodfacts.org
          <template v-slot:action>
            <q-btn>Cart <q-badge color="orange" floating>{{cart.length}}</q-badge></q-btn>
          </template>
        </q-banner>


<div class="q-pa-md row items-start q-gutter-md">
    <q-card v-for="(product, index) in products" class="my-card" v-bind:key="index">
      <q-img height="300" width="300" :src="product.image_front_small_url">
        <div class="absolute-bottom">
          <div class="text-h6">{{product.product_name}}</div>
          <div class="text-subtitle2">by John Doe</div>
        </div>
      </q-img>

      <q-card-actions>
        <q-btn @click="addToCart(product)" flat>Add Cart</q-btn>
      </q-card-actions>
    </q-card>
</div>
  </div>
</template>

<script>
export default {
  async mounted() {
    var requestOptions = {
      method: "GET",
      redirect: "follow",
    };
    console.log(this.$router.currentRoute)
    const product = this.$router.currentRoute?.value?.params?.category
    const catTitle = product.split(":")[1]

    const response = await fetch(
      `https://us.openfoodfacts.org/category/${catTitle}.json`,
      requestOptions
    );
    const data = await response.json();
    this.products = data.products.slice(0, 100)
  },
  data: () => ({
    products: null, newCartItem: null
  }),
  computed: {
      cart() {
          const component = this
          const newCartItem = this.newCartItem
          try {
            const cart = localStorage.getItem("cart")
            const cartItems = JSON.parse(cart)
            if(newCartItem) {
                if(cartItems.find(function(item){ return item.name == newCartItem.name}) == undefined)  cartItems.push(newCartItem)
            }
            component.newCartItem = null
            return cartItems || []
          } catch(e) {
              return []
          }
      }
  },
  setup() {
    return {
      lorem:
        "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.",
    };
  },
  methods: {
      async goToproduct(product) {
          this.$router.push(`/products/${product.id}`)
         // this.$router
      },
      async addToCart(product) {
          const cart = this.cart
          console.log("cart", cart)
          const basicProduct = {name: product.product_name}
          cart.push(basicProduct)
          this.newCartItem = basicProduct
          console.log("this is the cart now", cart)
          if(cart.find(function(item){ return item.name == this.newCartItem.name}) == undefined) localStorage.setItem("cart", JSON.stringify(cart))
        
      }
  }
};
</script>

<style lang="sass" scoped>
.my-card
    width: 100%
    max-width: 250px
</style>
