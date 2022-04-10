<template>
  <div>
          <q-banner class="bg-primary text-white">
          Start Capture
          <template v-slot:action>
              Powered by https://world.openfoodfacts.org/ and https://.openfoodfacts.org
            <q-btn >Take Picture</q-btn>
          </template>
        </q-banner>


<div class="q-pa-md row items-start q-gutter-md">
    <q-card v-for="(category, index) in categories" class="my-card" v-bind:key="index">
      <q-card-section class="bg-grey-8 text-white">
        <div class="text-h6">{{category.name}}</div>
      </q-card-section>

      <q-card-actions vertical align="center">
        <q-btn @click="goToCategory(category)" flat>Find Food in this category</q-btn>
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

    const response = await fetch(
      "https://us.openfoodfacts.org/categories.json",
      requestOptions
    );
    const data = await response.json();
    this.categories = data.tags.slice(0, 100);
  },
  data: () => ({
    categories: null,
  }),
  setup() {
    return {
      lorem:
        "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.",
    };
  },
  methods: {
      async goToCategory(category) {
          this.$router.push(`/categories/${category.id}`)
         // this.$router
      }
  }
};
</script>

<style lang="sass" scoped>
.my-card
    width: 100%
    max-width: 250px
</style>
