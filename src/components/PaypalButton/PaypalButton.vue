<template>
    <div v-if="isPaypalLoaded">
        Paypal Button
        <div id="paypal-button-container"></div>
    </div>
</template>

<script>
export default {
  name: 'vue-paypal-v2-button',
  props: {
    config: {
      type: Object,
      default: () => ({ client_id: 'sb', language: 'es', currency: 'USD' }),
    },
    buyer: {
      type: Object,
      default: () => ({}),
    },
    discount: {
      type: Number,
      default: 0,
    },
    shipping: {
      type: Number,
      default: 0,
    },
    invoiceId: {
      type: String,
      default: '',
    },
    items: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      error: '',
      isPaypalLoaded: false,
    };
  },
  created() {
    const self = this;
    // As an instance method inside a component
    if (typeof window.paypal === 'undefined') {
      this.$loadScript(`https://www.paypal.com/sdk/js?client-id=${this.config.client_id}&locale=${this.locale}&currency=${this.config.currency}`)
        .then(() => {
          this.isPaypalLoaded = true;
          self.$nextTick()
            .then(() => {
              self.load();
            });
        })
        .catch(() => {
          // Failed to fetch script
          self.$emit('error', 'Paypal Script Load Failed');
        });
    } else {
      this.isPaypalLoaded = true;
      self.$nextTick(() => {
        self.load();
      });
    }
  },
  computed: {
    locale() {
      let tmp;
      switch (this.config.language) {
        case 'es':
          tmp = 'es_ES';
          break;
        case 'en':
          tmp = 'en_US'; break;
        case 'fr':
          tmp = 'fr_US'; break;
        case 'it':
          tmp = 'it_IT'; break;
        default:
          tmp = 'en_US';
      }
      return tmp;
    },
    itemsTotal() {
      const sumOfItems = this.items.reduce((sum, currentValue) => sum + (currentValue.unit_amount * currentValue.quantity), 0);
      return (sumOfItems).toFixed(2);
    },
    totalAmount() {
      return (this.itemsTotal - this.discount).toFixed(2);
    },
    paypalItems() {
      const self = this;
      return this.items.map((item) => {
        const newItem = { ...item };
        newItem.unit_amount = {
          currency_code: self.config.currency,
          value: item.unit_amount,
        };
        return newItem;
      });
    },
  },
  methods: {
    load() {
      console.log('Element', document.querySelector('#paypal-button-container'));
      const self = this;
      window.paypal.Buttons({
        style: {
          label: 'pay',
        },
        onError(err) {
          console.log(err);
          self.$emit('error', err);
        },
        onApprove(data, actions) {
          // Capture the funds from the transaction
          return actions.order.capture().then((details) => {
            // Show a success message to your buyer
            self.$emit('success', details);
          });
        },
        createOrder(data, actions) {
          // Set up the transaction
          return actions.order.create({
            invoice_id: self.invoiceId,
            locale: self.config.language,
            purchase_units: [{
              items: self.paypalItems,
              amount: {
                currency_code: self.config.currency,
                value: self.totalAmount,
                breakdown: {
                  item_total: {
                    currency_code: self.config.currency,
                    value: self.itemsTotal,
                  },
                  shipping: {
                    currency_code: self.config.currency,
                    value: self.shipping,
                  },
                  discount: {
                    currency_code: self.config.currency,
                    value: self.discount,
                  },
                },
              },
            }],
          });
        },
      }).render('#paypal-button-container');
    },
  },
};
</script>

<style lang="scss" scoped>
</style>
