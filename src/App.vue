<template>
  <div id="app">
    <h1>Olo Pizza Exercise using Vue 3 Composition API and Vite</h1>
    <table class="comboCounts">
      <tr>
        <th>Rank</th>
        <th>Toppings</th>
        <th># times ordered</th>
      </tr>
      <tr
        :key="rank"
        v-for="{ rank, toppings, count } in topPizzaToppingComboCounts"
      >
        <td>{{ rank }}</td>
        <td>{{ toppings.join(", ") }}</td>
        <td>{{ count }}</td>
      </tr>
    </table>
  </div>
</template>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #eee;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.comboCounts {
  border-collapse: collapse;

  th,
  td {
    text-align: left;
    border: solid 1px #eee;
    padding: 0.5em;
  }

  th {
    vertical-align: bottom;
    background-color: #aaa;
  }

  td {
    vertical-align: top;
  }
}
</style>

<script setup>
import axios from "axios";
import { isEqual, orderBy } from "lodash";
import { onMounted, ref } from "vue";

document.title = "Olo Pizza Exercise";

const topPizzaToppingComboCounts = ref([]);

onMounted(async () => {
  // Using local proxy to get around CORS headers issue:
  // https://www.npmjs.com/package/cors-anywhere
  const { data: pizzas } = await axios.get(
    "http://localhost:8080/https://www.olo.com/pizzas.json"
  );

  const toppingComboCounts = pizzas.reduce(
    (toppingComboCounts, { toppings }) => {
      const normalizedToppings = toppings.slice().sort();
      const combo = toppingComboCounts.find(({ toppings: existingToppings }) =>
        isEqual(existingToppings, normalizedToppings)
      );

      if (combo) combo.count += 1;
      else toppingComboCounts.push({ toppings, count: 1 });

      return toppingComboCounts;
    },
    []
  );

  topPizzaToppingComboCounts.value = orderBy(
    toppingComboCounts,
    "count",
    "desc"
  )
    .slice(0, 20)
    .map((comboCount, index) => ({ ...comboCount, rank: index + 1 }));
});
</script>
