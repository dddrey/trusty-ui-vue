<template lang="pug">
#trusty_transfer

  ._turnover_inputs
    AlphaInput(label="enter sum" composed=true)
      template(slot="input")
        input(v-model="amount" ref="amount" @input="$v.amount.$touch()")
      template(slot="right")
        select(v-model="selectedCoin" v-if="isNonZeroLength" dir="rtl")
          option(v-for="(coin, id) in nonZeroAssets", v-bind:value="id") {{ coin.symbol }}
        icon-component(name="trusty_arrow_down")
    .trusty_font_error(v-if="!$v.amount.required && this.$v.amount.$dirty") Enter amount
    .trusty_font_error(v-if="$v.amount.required && !$v.amount.isNumeric && this.$v.amount.$dirty") Enter a number
    .trusty_font_error(v-if="$v.amount.isNumeric && !$v.amount.doesntExceedBalance && this.$v.amount.$dirty" @click="setAmount") {{ insufficientAmountText }}
    .trusty_font_error(
      v-if=`$v.amount.isNumeric &&
      $v.amount.doesntExceedBalance &&
      !$v.amount.doesntExceedMinWithdraw &&
      this.$v.amount.$dirty`) Minimal withdraw amount {{ minWithdraw }}

    p.withdraw_tooltip(v-if="!$v.amount.required" @click="setAmount") {{ balanceAmountText }}
    p
    TrustyInput(
      :isOpen="true",
      label="payment method",
      className="select_input payment-method",
      :close="false",
      :foreignInput="true")

      template(slot="input")
        icon-component(name="trusty_arrow_down" style="position: absolute")
        select(v-model="paymentMethod" )
          option(v-for="method in methods", :value="method") {{ method }}

  ._turnover_service
    component(:is="gateway", :amount="payload.amount", :coin="payload.selectedcoin")

</template>

<script>
import { mapGetters, mapActions } from 'vuex';
import { validationMixin } from 'vuelidate';
import { required } from 'vuelidate/lib/validators';
import AlphaInput from '@/components/UI/form/alpha';
import TrustyInput from '@/components/UI/form/input';
import iconComponent from '@/components/UI/icon';
import openledger from './Openledger/Withdraw';
import bitshares from './Bitshares/Withdraw';
import trusty from './Trusty/Withdraw';
import './style.scss';

const OpenledgerName = 'OpenLedger crypto gateway';
const BitsharesName = 'BitShares direct transfer';
const TrustyMethods = [
  'Tinkoff card2card transfer',
  'Sberbank card2card transfer',
  'VTB card2card transfer',
  'Alfa bank card2card transfer',
  'Raiffeisen card2card transfer',
  'VTB card2card transfer',
  'Rocket card2card transfer'
];

const methodsByGate = {
  openledger: [OpenledgerName],
  bitshares: [BitsharesName],
  trusty: TrustyMethods
};

export default {
  props: {
    coin: {
      type: String,
      required: false,
      default: '1.3.0'
    },
  },
  data() {
    return {
      selectedCoin: this.coin,
      paymentMethod: BitsharesName,
      amount: '',
    };
  },
  components: { TrustyInput, AlphaInput, iconComponent, openledger, bitshares, trusty },
  mixins: [validationMixin],
  validations: {
    amount: {
      required,
      isNumeric(value) {
        const parsed = +value;
        return parsed && !Number.isNaN(parsed);
      },
      doesntExceedBalance(value) {
        const id = this.selectedCoin;
        const balance = this.balances[id].balance / (10 ** this.getAssetById(id).precision);
        return value <= balance;
      },
      doesntExceedMinWithdraw(value) {
        if (this.paymentMethod === OpenledgerName) {
          return value > this.minWithdraw;
        }
        return true;
      }
    }
  },
  beforeMount() {
    this.fetchCoins();
    this.fetchBtcPrice();
  },
  watch: {
    selectedCoin() {
      this.amount = '';
    }
  },
  methods: {
    ...mapActions({
      fetchCoins: 'openledger/fetchCoins',
      fetchBtcPrice: 'cryptobot/fetchBtcPrice'
    }),
    setAmount() {
      const id = this.selectedCoin;
      // eslint-disable-next-line
      const precision = this.getAssetById(id).precision;
      const newAmount = this.balances[id].balance / (10 ** precision);
      this.amount = newAmount.toFixed(precision);
      this.$refs.amount.focus();
    },
    getAssetById(id) {
      if (id === 'RUB') {
        return { symbol: 'RUB', id: 'RUB', precision: 0 };
      }
      return this.getAsset(id);
    }
  },
  computed: {
    ...mapGetters({
      userBalances: 'account/getCurrentUserBalances',
      getAsset: 'assets/getAssetById',
      coinsData: 'openledger/getCoinsData',
      btcPrice: 'cryptobot/getBtcPrice'
    }),
    balances() {
      const balances = this.userBalances;
      const btcBalance = balances['1.3.861'];
      const rubBalance = { asset_type: 'RUB', balance: 1 };
      if (btcBalance) {
        rubBalance.balance = Math.floor((this.btcPrice * btcBalance.balance) / (10 ** 8));
      }
      balances.RUB = rubBalance;
      return balances;
    },
    balanceAmountText() {
      const id = this.selectedCoin;
      // eslint-disable-next-line
      const precision = this.getAssetById(id).precision;
      const balance = this.balances[id].balance / (10 ** precision);
      return 'max ' + balance.toFixed(precision) + ' (click to paste)';
    },
    insufficientAmountText() {
      const id = this.selectedCoin;
      const { symbol, precision } = this.getAssetById(id);
      const balance = this.balances[id].balance / (10 ** precision);
      const prefixText = 'insufficient funds, max ';
      return prefixText + balance.toFixed(precision) + ' ' + symbol + '(click to paste)';
    },
    minWithdraw() {
      if (this.paymentMethod === OpenledgerName) {
        const coin = this.getAssetById(this.selectedCoin);
        const coinName = coin.symbol.toLowerCase();
        const { gateFee } = this.coinsData[coinName];
        return parseFloat(gateFee) * 2;
      }
      return 0;
    },
    isNonZeroLength() {
      return Object.keys(this.nonZeroAssets).length;
    },
    nonZeroAssets() {
      const result = {};
      Object.keys(this.balances).forEach(id => {
        if (this.balances[id].balance) {
          result[id] = this.getAssetById(id);
        }
      });
      return result;
    },
    nonZeroBalanceAssetsIds() {
      return Object.keys(this.balances)
        .filter(id => this.balances[id].balance)
        .map(id => {
          const { symbol } = this.getAssetById(id);
          return { symbol, id };
        });
    },
    transferConfig() {
      return {
        openledger: [],
        transfer: this.nonZeroBalanceAssetsIds
      };
    },
    gateway() {
      let selectedGateway = false;
      Object.keys(methodsByGate).forEach((gateway) => {
        if (methodsByGate[gateway].includes(this.paymentMethod)) {
          selectedGateway = gateway;
        }
      });
      return selectedGateway;
    },
    methods() {
      const availableMethods = [BitsharesName];
      const { issuer } = this.getAssetById(this.selectedCoin);
      if (issuer === '1.2.96397') {
        availableMethods.push(OpenledgerName);
      }

      if (this.selectedCoin === 'RUB') {
        [this.paymentMethod] = TrustyMethods;
        return TrustyMethods;
      }

      [this.paymentMethod] = availableMethods;
      return availableMethods;
    },
    currentAssetAmount() {
      if (this.$v.$invalid) return 0;
      return Math.floor(this.amount * (10 ** this.getAssetById(this.selectedCoin).precision));
    },
    payload() {
      return {
        selectedcoin: this.selectedCoin,
        amount: this.currentAssetAmount
      };
    }

  }
};

</script>

<style lang="scss">
._input_space.composed {
  width: 68vw!important;
}
.withdraw_tooltip{
  font-size: 3.3vw;
  font-family: "Gotham_Pro";
  color: white;
  margin: 0;
  padding: 0;
}
</style>
