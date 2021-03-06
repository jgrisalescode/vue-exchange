<template>
  <div class="flex-col">
    <div class="flex justify-center">
      <bounce-loader
        :loading="isLoading"
        :color="'#68d391'"
        :size="100"
      ></bounce-loader>
    </div>
    <template v-if="!isLoading">
      <div class="flex flex-col sm:flex-row justify-around items-center">
        <div class="flex flex-col items-center">
          <img
            v-bind:src="
              `https://static.coincap.io/assets/icons/${asset.symbol.toLowerCase()}@2x.png`
            "
            v-bind:alt="asset.name"
            class="w-20 h-20 mr-5"
          />
          <h1 class="text-5xl">
            {{ asset.name }}
            <small class="sm:mr-2 text-gray-500">{{ asset.symbol }}</small>
          </h1>
        </div>

        <div class="my-10 flex flex-col">
          <ul>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Ranking</b>
              <span>#{{ asset.rank }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Current price</b>
              <span>{{ asset.priceUsd | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Lowest price</b>
              <span>{{ min | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Highest price</b>
              <span>{{ max | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Average price</b>
              <span>{{ avg | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Variation 24hs</b>
              <span>{{ asset.changePercent24Hr | percent }}</span>
            </li>
          </ul>
        </div>

        <div class="my-10 sm:mt-0 flex flex-col justify-center text-center">
          <button
            v-on:click="toggleConverter"
            class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded"
          >
            {{ fromUsd ? `USD to ${asset.symbol}` : `${asset.symbol} to USD` }}
          </button>

          <div class="flex flex-row my-5">
            <label class="w-full" for="convertValue">
              <input
                v-model="convertValue"
                id="convertValue"
                type="number"
                class="text-center bg-white focus:outline-none focus:shadow-outline border border-gray-300 rounded-lg py-2 px-4 block w-full appearance-none leading-normal"
                v-bind:placeholder="
                  `Value on ${fromUsd ? 'USD' : asset.symbol}`
                "
              />
            </label>
          </div>
          <span class="text-xl"
            >{{ convertResult }} {{ fromUsd ? asset.symbol : "USD" }}</span
          >
        </div>
      </div>
      <line-chart
        v-bind:colors="['orange']"
        :min="min"
        :max="max"
        :data="
          history.map(item => [item.date, parseFloat(item.priceUsd).toFixed(2)])
        "
        class="my-10"
      />
      <h3 class="text-xl my-10">Best Exchange Offers</h3>
      <table>
        <tr
          v-for="m in markets"
          :key="`${m.exchangeId}-${m.priceUsd}`"
          class="norder-b"
        >
          <td>
            <b>{{ m.exchangeId }}</b>
          </td>
          <td>{{ m.priceUsd | dollar }}</td>
          <td>{{ m.baseSymbol }} / {{ m.quoteSymbol }}</td>
          <td>
            <px-button
              v-bind:is-loading="m.isLoading || false"
              v-if="!m.url"
              v-on:click="getWebSite(m)"
            >
              <slot>Get Link</slot>
            </px-button>
            <a
              v-else
              v-bind:href="m.url"
              class="hover:underline text-green-600"
              target="_blank"
              >{{ m.url }}</a
            >
          </td>
        </tr>
      </table>
    </template>
  </div>
</template>
<script>
import api from "@/api/index.js";
import PxButton from "../components/PxButton.vue";
export default {
  components: { PxButton },
  name: "CoinDetail",
  data() {
    return {
      asset: {},
      history: [],
      isLoading: false,
      markets: [],
      fromUsd: true,
      convertValue: null
    };
  },

  computed: {
    convertResult() {
      if (!this.convertValue) {
        return 0;
      }

      const result = this.fromUsd
        ? this.convertValue / this.asset.priceUsd
        : this.convertValue * this.asset.priceUsd;

      return result.toFixed(4);
    },

    min() {
      return Math.min(
        ...this.history.map(h => parseFloat(h.priceUsd).toFixed(2))
      );
    },

    max() {
      return Math.max(
        ...this.history.map(h => parseFloat(h.priceUsd).toFixed(2))
      );
    },

    avg() {
      return Math.abs(
        ...this.history.map(h => parseFloat(h.priceUsd).toFixed(2))
      );
    }
  },

  watch: {
    $route() {
      this.getCoin();
    }
  },

  created() {
    this.getCoin();
  },

  methods: {
    toggleConverter() {
      this.fromUsd = !this.fromUsd;
    },
    getWebSite(exchange) {
      return api
        .getExchange(exchange.exchangeId)
        .then(res => {
          this.$set(exchange, "isLoading", true);
          // exchange.url = res.exchangeUrl;
          /*
        Wen we click on button Vue does not detect the change, so solve with
        Form the beginin the object market does not have a prop url, then wen we
        click on button, the prop has been created inside object, but Vue does not detect
        the change, we have to solve it with this below
        */
          this.$set(exchange, "url", res.exchangeUrl);
        })
        .finally(() => this.$set(exchange, "isLoading", false));
    },
    getCoin() {
      // get the parameter setted in router for this route
      const id = this.$route.params.id;
      this.isLoading = true;
      Promise.all([
        api.getAsset(id),
        api.getAssetHistory(id),
        api.getMarkets(id)
      ])
        .then(([asset, history, markets]) => {
          this.asset = asset;
          this.history = history;
          this.markets = markets;
        })
        .finally(() => (this.isLoading = false));
    }
  }
};
</script>
<style scoped>
td {
  padding: 10px;
  text-align: center;
}
</style>
