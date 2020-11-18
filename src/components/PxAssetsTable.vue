<template>
  <table>
    <thead>
      <tr class="bg-gray-100 border-b-2 border-gray-400">
        <th></th>
        <th>
          <span>Ranking</span>
        </th>
        <th>Name</th>
        <th>Price</th>
        <th>Market capital</th>
        <th>Variation 24hs</th>
        <td class="hidden sm:block"></td>
      </tr>
    </thead>
    <tbody>
      <tr
        v-for="a in assets"
        v-bind:key="a.id"
        class="border-b border-gray-200 hover:bg-gray-100 hover:bg-orange-100"
      >
        <td>
          <img
            class=".max-w-xs"
            v-bind:src="
              `https://static.coincap.io/assets/icons/${a.symbol.toLowerCase()}@2x.png`
            "
            v-bind:alt="a.name"
          />
        </td>
        <td>
          <b>#{{ a.rank }}</b>
        </td>
        <td>{{ a.name }}</td>
        <td>{{ a.priceUsd }}</td>
        <td>{{ a.marketCapUsd }}</td>
        <td>{{ a.changePercent24Hr }}</td>
        <td class="hidden sm:block"></td>
      </tr>
    </tbody>
  </table>
</template>
<script>
export default {
  name: "PxAssetsTable",
  props: {
    assets: {
      type: Array,
      defaul: () => []
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
