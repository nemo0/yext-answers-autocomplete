<script setup>
import { ref, watchEffect } from "vue";
import { provideCore } from "@yext/answers-core";

const core = provideCore({
  apiKey: "4be28826989e90232722e9bf2769fbf2",
  experienceKey: "cpg-beverages",
  locale: "en",
  experienceVersion: "PRODUCTION",
});

let searchTerm = ref("");
let result = ref([]);
let selectedResult = ref("");

const searchResults = watchEffect(async () => {
  if (searchTerm.value === "") {
    return [];
  }
  result.value = await core.universalAutocomplete({
    input: searchTerm.value,
  });
  document.getElementById("results").style.display = "block";
});
const selectResult = async (result) => {
  searchTerm.value = "";
  const r = await core.universalSearch({
    query: result,
  });
  console.log(r);
  let ansArr = [];
  r.verticalResults.reduce((acc, cur) => {
    cur.results.forEach((res) => {
      if (res.description !== undefined) {
        acc.push(res);
      }
    });
    return acc;
  }, ansArr);
  selectedResult.value = ansArr;
  console.log(ansArr);
  document.getElementById("results").style.display = "none";
};
</script>

<template>
  <div class="bg-gray-50 min-w-screen min-h-screen flex justify-center py-10">
    <div class="max-w-lg relative space-y-3 text-center">
      <h3 class="font-extrabold text-lg">Universal Autocomplete</h3>
      <label for="search" class="text-gray-900">
        Type the name of a data to search(examples: "how", "cider", etc.)
      </label>

      <input
        type="text"
        id="search"
        v-model="searchTerm"
        placeholder="Type here..."
        class="p-3 mb-0.5 w-full border border-gray-300 rounded"
      />

      <ul
        v-if="result.results !== undefined"
        class="
          w-full
          rounded
          bg-white
          border border-gray-300
          px-4
          py-2
          space-y-1
          absolute
          z-10
          drop-shadow-md
        "
        id="results"
      >
        <li class="px-1 pt-1 pb-2 font-bold border-b border-gray-200">
          Showing {{ result.results.length }} results
        </li>
        <li
          v-for="r in result.results"
          :key="r.id"
          @click="selectResult(r.value)"
          class="cursor-pointer hover:bg-gray-100 p-1"
        >
          {{ r.value }}
        </li>
      </ul>
      <ul
        v-if="selectedResult.length"
        class="text-lg pt-2 z-0 grid grid-cols-1 md:grid-cols-2"
      >
        <li
          v-for="r in selectedResult"
          :key="r.id"
          class="
            inline-block
            bg-gray-200
            rounded-md
            px-3
            py-1
            text-gray-700
            mr-2
            mb-2
          "
        >
          <a class="text-lg font-semibold underline text-blue-600" href="#">{{
            r.name
          }}</a>
          <div v-html="r.description"></div>
        </li>
      </ul>
    </div>
  </div>
</template>

