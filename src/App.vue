<template>
  <div id="app">
    <h2>Button</h2>
    <paypal-button v-if="show" :items="items" :buyer="buyer" @success="data=$event"  @error="data=$event"/>
    <button @click="show=true">Mostrar</button>
    <button @click="show=false">Ocultar</button>
    <div class="bg-red-500 border-red-100">{{data}}</div>
  </div>
</template>

<script>
import PaypalButton from './components/PaypalButton';

export default {
  name: 'app',
  components: {
    PaypalButton,
  },
  data() {
    return {
      show: true,
      data: '',
      buyer: {
        name: 'Sapo Pepe',
        address: 'Siempre Vivi 200',
        countryCode: 'AR',
        city: 'Buenos Aires',
        zipCode: '1425',
      },
      items: [
        {
          name: 'Test 1', unit_amount: 10, quantity: 1, sku: 1,
        },
        {
          name: 'Test 2', unit_amount: 9.90, quantity: 2, sku: 2,

        },
        {
          name: 'Test 3', unit_amount: 7.77, quantity: 3, sku: 3,
        },
      ],
    };
  },
  created() {},
  mounted() {
  },
};
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
