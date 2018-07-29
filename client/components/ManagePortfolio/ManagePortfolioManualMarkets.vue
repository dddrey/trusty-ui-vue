<template lang="pug">
.manual_trading_markets
  template(v-for="market in markets")
    div(@click="goToMarket(market)") {{ getAssetById(market).symbol }}
</template>

<script>
import { mapGetters } from 'vuex';

export default {
  props: ['markets'],
  data() {
    return {};
  },
  computed: {
    ...mapGetters({
      getAssetById: 'assets/getAssetById'
    })
  },
  methods: {
    goToMarket(market) {
      this.$router.push({ name: 'manage-manual-market', params: { market } });
    }
  }
};
</script>

<style>
.manual_trading_markets {
  display: grid;
  grid-template-rows: 50vw 50vw;
  grid-template-columns: 50vw 50vw;
  justify-items: center;
  align-items: center;
  color: white;
}
</style>