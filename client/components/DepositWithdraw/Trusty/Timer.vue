<template lang="pug">
.trusty_deposit_fiat_fullscreen
  .trusty_deposit_timer
    ._title(v-html="text" v-bind:class='{"error": error || textChange}')
    ._timer(v-if="error")
      | 00:00
    ._timer(v-else)
      | 0{{minutesRemaining}}
      span
        | :
        span {{ secondsString }}
    .trusty_inline_buttons._one_button
      button(@click="buttonAction") {{ buttonText }}

</template>

<script>
import { mapActions } from 'vuex';

export default {
  props: {
    error: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      secondsRemaining: 59,
      minutesRemaining: 2
    };
  },
  methods: {
    ...mapActions({
      cancelOrder: 'cryptobot/cancelOrder',
      clearOrder: 'cryptobot/clearOrder',
      hideHeader: 'app/hideHeader',
      showHeader: 'app/showHeader'
    }),

    tick() {
      let newMinutes = this.minutesRemaining;
      let newSeconds = this.secondsRemaining - 1;

      if (newSeconds % 60 === 0 && newMinutes !== 0) {
        newMinutes = this.minutesRemaining - 1;

        if (newMinutes >= 0) {
          newSeconds = 60;
        }
      }

      this.secondsRemaining = newSeconds;
      this.minutesRemaining = newMinutes;

      if (this.secondsRemaining <= 0 && this.minutesRemaining <= 0) {
        clearInterval(this.interval);
      }
    },
    buttonAction() {
      if (this.error) {
        this.clearOrder();
      } else {
        this.cancelOrder();
      }
      this.$nextTick(() => {
        window.scrollTo(0, 0);
      });
    }
  },

  computed: {
    secondsString() {
      const seconds = this.secondsRemaining;
      return (this.secondsRemaining < 10) ? '0' + seconds : seconds;
    },
    textChange() {
      return this.secondsRemaining === 0 && this.minutesRemaining === 0;
    },
    text() {
      if (this.error) {
        return 'PLEASE TRY AGAIN LATER';
      }
      if (this.textChange) {
        return 'PLEASE WAIT MORE <br /> WE ARE HANDLING YOUR REQUEST';
      }
      return 'YOU WILL GET DEPOSIT DETAILS IN <br /> UNDER 3 MINUTES';
    },
    buttonText() {
      if (this.error) {
        return 'Back';
      }
      return 'Cancel order';
    }
  },

  mounted() {
    this.interval = setInterval(this.tick.bind(this), 1000);
    this.hideHeader();
  },
  beforeDestroy() {
    this.showHeader();
    clearInterval(this.interval);
  }

};

</script>

<style lang="scss" >



@media screen and (max-width: 768px) {

  .trusty_deposit_timer {
    color: white;
    text-align: center;
    ._title {
      font-size: 4.4vw !important;
        font-family: "Gotham_Pro_Regular";
        line-height: 5.5vw;
    }

    .error {
      color: #fdf101;
    }

    ._timer {
      margin: 4vw;
      font-family: "Gotham_Pro";
      font-size: 17vw;    }
  }

}

</style>

