 <template lang="pug">
div
	div.trusty_fixed_background_div
	div#landing
		div.balls_nav._desk
			span(
				v-for="refer in slideRefers",
				@click="scrollTo(refer)",
				:class="{ _selected: referClass === refer }")
		div.logo_starter(:style="{ height: windowHeight }")
			div.top_buttons
				a(@click="clickLink('signup')")
					span SignUp
				a(@click="clickLink('faq')")
					span FAQ
				a(@click="clickLink('login')")
					span LogIn

			div.bottom_content
				div._logo_text._mob(v-html="logo")
				div.wrap_img_profile._desk
					img(src='./vendor/img_user_page.png')
				div._desk_right
					div._logo._desk(v-html="logoDesk")
					div._logo_text._desk(v-html="logo")
					p._slogan Investment Wallet
					p._description
						| One-Click To Invest In&nbsp
						br._mob
						| Crypto Economy
					a._desk
						button.land(@click="clickLink('signup')") INVEST NOW
			div._fixed_bottom._mob
				a
					button.land(@click="clickLink('signup')") INVEST NOW
				div.trusty_down_arrow(@click="clickScroll(0)")
					span(v-html="arrowDown")

		div.land_slides
			template(v-for="(slide, index) in slides")
				section.land_slide(
					:style="slideHeight"
					:class="slideClass(index)",
					:id="slideClass(index)",
					:ref="slideClass(index)",
					@click="clickScroll(index + 1)")
					div.image_area
						div
							template(v-if="index === 0")
								.phontom_buttons
									span(
										v-for="refer in iconsRefers",
										@click.stop="scrollTo(refer)")
								img._image._mob(:src="slide.image")
								img._image._desk(:src="slide.imageDesk")
							template(v-else)
								img._image(:src="slide.image")
					div.text_area
						h1(v-html="slide.title")
						div._body(v-if="slide.text" v-html="slide.text")
					div.trusty_down_arrow
						span(v-html="arrowDown")

		div.last_text#last_screen(ref="last")
			p
				| First time in history&nbsp
				br._mob
				| everybody
				br._desk
				|   can invest&nbsp
				br._mob
				|  in a globally disruptive,
				br
				| yet infant, technology
			a.wrap_button
				button.land(@click="clickLink('signup')") INVEST NOW
			p
				| Depositing into Trusty.Fund
				br._mob
				|  now is like
				br._desk
				|  early investing
				br._mob
				|  in internet companies,
				br._desk
				|  when 20 million people
				br._mob
				|  used internet
			a.wrap_button
				button.land(@click="clickLink('faq')") FAQ

		div._land_bottom_info._desk
			section
				| The above references are for information purposes only.
				br._mob
				|  They are not intended to be investment advices.
				br
				| Trusty.Fund provides a trustless service for its clients to manage and
				br._mob
				| store funds on the BitShares blockchain via trusted gateways.
			section 2018 Trusty.Fund

	div._land_bottom_info._mob
		section
			| The above references are for information purposes only.
			br._mob
			|  They are not intended to be investment advices.
			br
			| Trusty.Fund provides a trustless service for its clients to manage and
			br._mob
			| store funds on the BitShares blockchain via trusted gateways.
		section 2018 Trusty.Fund

</template>

<script>
import { mapGetters } from 'vuex';
import { isMobile } from './utils';
import './style.scss';

const slide1 = require('./vendor/how.gif');
const slide1Desk = require('./vendor/how_desk.gif');
const slide2 = require('./vendor/network.gif');
const slide3 = require('./vendor/wallet1.gif');
const slide4 = require('./vendor/img_1click_to_invest.png');
const slide5 = require('./vendor/img_chng_portfolio.png');
const slide6 = require('./vendor/img_index.png');
const slide7 = require('./vendor/wallet2.gif');
const arrowDown = require('./vendor/trusty_arrow_down.svg');
const logo = require('./vendor/img_trusty_logo_last.svg');
const logoDesk = require('./vendor/logo.svg');

export default {
  data() {
    return {
      slides: [
        {
          image: slide1,
          imageDesk: slide1Desk,
          title: 'How To Use',
          text: null
        },
        {
          image: slide2,
          title: "Easy To Create<br class='_desk'> A<br class='_mob'>&nbsp" +
"Decentralized<br class='_desk'> Account",
          text: 'Click Signup, create password<br> and your account will be secured <br>' +
'by the BitShares.org blockchain. <br> You own the private key'
        },
        {
          image: slide3,
          title: 'Deposit Fiat<br>Or Cryptocurrencies',
          text: 'Invest USD, RUB, EUR, CNY at the best exchange rate or pay 0% ' +
'commission to deposit cryptocurrencies directly'
        },
        {
          image: slide4,
          title: 'One-Click To<br> Buy A Portfolio Of Cryptos',
          text: 'After deposit, you are a click<br> away from your own customized<br>' +
'portfolio of crypto assets.<br> Forget the hassle of buying<br> assets separately'
        },
        {
          image: slide5,
          title: "One-Click<br>To Manage<br class='_desk'> Your Portfolio",
          text: 'Manage your portfolio by mirroring<br> trades of ranked portfolio<br> managers, ' +
'applying index rules or<br> using the portfolio rebalancing tool'
        },
        {
          image: slide6,
          title: "One-Click To<br class='_desk'>" +
" Fix <br class='_mob'> Your Income<br class='_desk'> In USD",
          text: 'Fix your income to wait out price<br> hyper volatility. Just click to<br> ' +
'transfer your funds in<br> USD, EUR, CNY, Gold, etc.'
        },
        {
          image: slide7,
          title: 'Withdraw Fiat<br> Or Cryptocurrencies',
          text: 'Withdraw funds in USD, RUB, EUR, CNY<br class="_desk" /> directly to a bank card' +
', payment service<br class="_desk" />  account or send cryptocurrencies' +
'<br class="_desk" />  to a crypto wallet'
        }
      ],
      arrowDown,
      logo,
      logoDesk,
      referClass: '',
      slideRefers: '',
      slideHeight: '',
      windowHeight: '',
      iconsRefers: '',
    };
  },
  computed: {
    ...mapGetters({
      authUser: 'account/getAccountUserId'
    })
  },
  mounted() {
    this.windowHeight = window.innerHeight + 'px';
    this.slideRefers = [1, 2, 3, 4, 5, 6, 7].map(item => `sl_id-${item}`);
    this.iconsRefers = this.slideRefers.slice(1);
    this.slideHeight = isMobile() ? { height: this.windowHeight } : {};
    window.addEventListener('scroll', this.handleScroll);
  },
  beforeDestroy() {
    window.removeEventListener('scroll', this.handleScroll);
  },
  methods: {
    handleScroll() {
      if (!isMobile()) {
        this.slideRefers.forEach(refer => {
          const el = this.$refs[refer][0];
          const rect = el.getBoundingClientRect();
          if (window.scrollY >= parseFloat(this.windowHeight)) {
            if (rect.top <= 0 && Math.abs(rect.top) <= el.clientHeight) {
              this.referClass = refer;
            }
          } else {
            this.referClass = '';
          }
        });
      }
    },
    slideClass(index) {
      const addString = index + 1;
      return 'sl_id-' + addString;
    },
    clickLink(destination) {
      if (this.authUser === null) {
        if (!isMobile()) {
					alert('Now available only on mobile devices'); // eslint-disable-line
        } else {
          this.$router.push({ name: destination });
        }
      } else {
        this.$router.push({ name: 'profile' });
      }
    },
    scrollTo(element) {
      let current;
      if (typeof element === 'string') {
        current = document.getElementById(element);
      } else {
        current = element;
      }
      this.$scrollTo(current, 500);
    },
    clickScroll(index) {
      if (index === this.slides.length) {
        this.scrollTo(this.$refs.last);
      } else {
        this.scrollTo(this.$refs[this.slideClass(index)][0]);
      }
    }
  }
};

</script>

<style scoped>
#landing {
	margin-top: -12vw;

}

</style>
