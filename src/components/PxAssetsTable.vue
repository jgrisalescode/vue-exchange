<template>
  <table>
    <thead>
      <tr class="bg-gray-100 border-b-2 border-gray-400">
        <th></th>
        <th
          v-bind:class="{ up: this.sortOrder == 1, down: this.sortOrder == -1 }"
        >
          <span v-on:click="changeSortOrder" class="underline cursor-pointer"
            >Ranking</span
          >
        </th>
        <th>Name</th>
        <th>Price</th>
        <th>Market capital</th>
        <th>Variation 24hs</th>
        <td class="hidden sm:block">
          <input
            type="text"
            name=""
            id="filter"
            placeholder="Search..."
            v-model="filter"
            class="bg-gray-100 focus:online-none border-b border-gray-400 py-2 px-4 block w-full appearance-none leading-normal"
          />
        </td>
      </tr>
    </thead>
    <tbody>
      <tr
        v-for="a in filteredAssets"
        v-bind:key="a.id"
        class="border-b border-gray-200 hover:bg-gray-100 hover:bg-orange-100"
      >
        <td>
          <img
            class="w-6 h-6"
            v-bind:src="
              `https://static.coincap.io/assets/icons/${a.symbol.toLowerCase()}@2x.png`
            "
            v-bind:alt="a.name"
          />
        </td>
        <td>
          <b>#{{ a.rank }}</b>
        </td>
        <td>
          <router-link
            class="hover:underline text-green-600"
            v-bind:to="{ name: 'coin-detail', params: { id: a.id } }"
          >
            {{ a.name }}
            <small class="ml-1 text-gray-500">
              {{ a.symbol }}
            </small>
          </router-link>
        </td>
        <td>{{ a.priceUsd | dollar }}</td>
        <td>{{ a.marketCapUsd | dollar }}</td>
        <td
          v-bind:class="
            a.changePercent24Hr.includes('-')
              ? 'text-red-600'
              : 'text-green-600'
          "
        >
          {{ a.changePercent24Hr | percent }}
        </td>
        <td class="hidden sm:block">
          <px-button v-on:click="goToCoin(a.id)">
            <span>Details</span>
          </px-button>
        </td>
      </tr>
    </tbody>
  </table>
</template>
<script>
import PxButton from "@/components/PxButton";
export default {
  name: "PxAssetsTable",
  components: {
    PxButton
  },
  data() {
    return {
      filter: "",
      sortOrder: 1
    };
  },
  props: {
    assets: {
      type: Array,
      defaul: () => []
    }
  },
  computed: {
    filteredAssets() {
      const altOrder = this.sortOrder == 1 ? -1 : 1;

      return this.assets
        .filter(
          asset =>
            asset.symbol.toLowerCase().includes(this.filter.toLowerCase()) ||
            asset.name.toLowerCase().includes(this.filter.toLowerCase())
        )
        .sort((a, b) => {
          if (parseInt(a.rank) > parseInt(b.rank)) {
            return this.sortOrder;
          }

          return altOrder;
        });
    }
  },
  methods: {
    goToCoin(id) {
      // Access to an instance of router this.$router
      this.$router.push({ name: "coin-detail", params: { id } });
    },
    changeSortOrder() {
      this.sortOrder = this.sortOrder == 1 ? -1 : 1;
    }
  }
};
</script>
<style scoped>
.up::before {
  content: "👆";
}

.down::before {
  content: "👇";
}

td {
  padding: 20px 0px;
  font-size: 0.6rem;
  text-align: center;
}

th {
  padding: 5px;
  font-size: 0.6rem;
}

@media (min-width: 640px) {
  td,
  th {
    padding: 20px;
    font-size: 1rem;
  }

  th {
    padding: 12px;
  }
}
</style>
