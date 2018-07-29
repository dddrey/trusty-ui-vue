<template lang="pug">
.manual_trading_market_asset_container
  span._back(@click='$router.go(-1)')
      icon-component(name="trusty_arrow_back")
  .manual_trading_market_asset 
    Spinner(size="small", absolute, v-if="isPending")
    .orders.sell_orders
      h1 Sell orders
      template(v-for="order in sellOrders")
        p {{ order.for_sale }} - {{ dumbPrice(order) }}
    .orders.buy_orders
      h1 Buy Orders
      template(v-for="order in buyOrders")
        p {{ order.for_sale }} - {{ dumbPrice(order) }}
</template>

<script>
import { mapGetters, mapActions } from 'vuex';
import Spinner from '@/components/UI/Spinner';
import iconComponent from '@/components/UI/icon';

export default {
  props: ['market', 'asset'],
  data() {
    console.log('Market Asset', this.market, this.asset);
    return {};
  },
  computed: {
    ...mapGetters({
      getAssetById: 'assets/getAssetById',
      getMarketOrders: 'market2/getMarketOrders',
      isPending: 'market2/isPending'
    }),
    buyOrders() {
      return this.getMarketOrders(this.market, this.asset).buy;
    },
    sellOrders() {
      return this.getMarketOrders(this.market, this.asset).sell;
    }
  },
  methods: {
    ...mapActions({
      hideHeader: 'app/hideHeader',
      showHeader: 'app/showHeader',
      subscribeToMarketOrders: 'market2/subscribeToMarketOrders',
      unsubscribeFromMarket: 'market2/unsubscribeFromMarket'
    }),
    dumbPrice(order) {
      return order.sell_price.base.amount / order.sell_price.quote.amount;
    }
  },
  created() {
    const baseId = this.market;
    const assetId = this.asset;
    this.subscribeToMarketOrders({ baseId, assetId });
    this.hideHeader();
  },
  beforeDestroy() {
    this.unsubscribeFromMarket({ baseId: this.market });
    this.showHeader();
  },
  components: {
    Spinner, iconComponent
  }
};
</script>

<style>
.manual_trading_market_asset_container {
  position: absolute;
  width: 100vw;
  height: 100vh;
  top: 0;
  left: 0;
}
.manual_trading_market_asset {
  display: grid;
  padding-top: 12vw;
  background-color: #1e2225;
  grid-template-rows: [first] calc(48vh - 12vw / 2) [middle] calc(48vh - 12vw / 2) [second];
  grid-template-columns: [first] 48vw [middle] 48vw [second];
  grid-row-gap: 2vh;
  grid-column-gap: 2vw;
}
.manual_trading_market_asset .orders {
  overflow-y: scroll;
}

.manual_trading_market_asset .sell_orders {
  grid-row-start: first;
  grid-row-end: middle;
  grid-column-start: middle;
  grid-column-end: second;
}

.manual_trading_market_asset .buy_orders {
  grid-row-start: middle;
  grid-row-end: second;
  grid-column-start: middle;
  grid-column-end: second;
}
</style>