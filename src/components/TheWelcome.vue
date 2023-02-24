<script setup lang="ts">
import axios from 'axios';
import {reactive} from 'vue'

const suffixes = [
  'age', 'al', 'ance', 'ence', 'dom', 'ee', 'er', 'or', 'hood', 'ism', 'ist', 'ity', 'ty', 'ment', 'ness', 'ry', 'ship', 'sion', 'tion', 'xion', 'able', 'ible', 'al', 'en', 'ese', 'ful', 'ic', 'ish', 'ive', 'ian', 'less', 'ly', 'ous', 'y', 'ify', 'ate', 'ise', 'ize', 'ward', 'wise'
];

const state = reactive({
  searchVal: '',
  results: [{}],
  suggestions: [''],
})

const generateNames = () => {
  let suggestions = [];
  for (let i = 0; i < 10; i++) {
    const searchTerms = state.searchVal.split(' ');
    const randomPrefix = searchTerms[Math.floor(Math.random() * searchTerms.length)];
    const randomSuffix = suffixes[Math.floor(Math.random() * suffixes.length)];
    const randomName = `${randomPrefix}${randomSuffix}`;
    suggestions.push(randomName);
  }

  for (let i = 0; i < 10; i++) {
    const randomPrefix = state.results[Math.floor(Math.random() * state.results.length)].word;
    const randomName = `${randomPrefix}`;
    suggestions.push(randomName);
  }

  for (let i = 0; i < 10; i++) {
    const randomPrefix = state.results[Math.floor(Math.random() * state.results.length)].word;
    const randomSuffix = suffixes[Math.floor(Math.random() * suffixes.length)];
    const randomName = `${randomPrefix}${randomSuffix}`;
    suggestions.push(randomName);
  }
  state.suggestions = suggestions;
}

const handleSearch = (event: { preventDefault: () => void; }) => {
  event.preventDefault();
  const searchTerms = state.searchVal.split(' ');
  if (searchTerms.length > 1) {
    searchTerms.forEach((term) => {
      axios.get(`https://api.datamuse.com/words?ml=${term}`)
          .then((response) => {
            state.results.push(...response.data);
          });
      axios.get(`https://api.datamuse.com/words?sl=${term}`)
          .then((response) => {
            state.results.push(...response.data);
          });
    })
    const searchTerm = state.searchVal.split(' ').join('+');
    axios.get(`https://api.datamuse.com/words?ml=${searchTerm}`)
        .then((response) => {
          state.results.push(...response.data);
          generateNames();
        });
  } else {
    axios.get(`https://api.datamuse.com/words?ml=${state.searchVal}`)
        .then((response) => {
          // handle success
          console.log(response);
          state.results.push(...response.data);
        });
    axios.get(`https://api.datamuse.com/words?sl=${state.searchVal}`)
        .then((response) => {
          // handle success
          console.log(response);
          state.results.push(...response.data);
          generateNames();
        });
  }

}

</script>

<template>
  <form class="flex">
    <label for="default-search" class="mb-2 text-sm font-medium text-gray-900 sr-only dark:text-white">Search</label>
    <div class="relative">
      <div class="absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none">
        <svg aria-hidden="true" class="w-5 h-5 text-gray-500 dark:text-gray-400" fill="none" stroke="currentColor"
             viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path>
        </svg>
      </div>
      <input v-model="state.searchVal" type="search" id="default-search"
             class="block w-full p-4 pl-10 text-sm text-gray-900 border border-gray-300 rounded-lg bg-gray-50 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
             placeholder="Search..." required>
      <button type="submit" @click="handleSearch"
              class="text-white absolute right-2.5 bottom-2.5 bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm px-4 py-2 dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800">
        Search
      </button>
    </div>
  </form>
  <div v-if="state.results.length > 1">
    <button @click="generateNames">Regenerate</button>
    <li v-for="suggestion in state.suggestions">
      {{ suggestion }}
    </li>
  </div>
</template>
