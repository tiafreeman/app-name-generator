<script setup lang="ts">
import axios from 'axios';
import {reactive} from 'vue'

const suffixes = [
  'age', 'al', 'ance', 'ence', 'dom', 'ee', 'er', 'or', 'hood', 'ism', 'ist', 'ity', 'ty', 'ment', 'ness', 'ry', 'ship', 'sion', 'tion', 'xion', 'able', 'ible', 'al', 'en', 'ese', 'ful', 'ic', 'ish', 'ive', 'ian', 'less', 'ly', 'ous', 'y', 'ify', 'ate', 'ise', 'ize', 'ward', 'wise'
];

const prefixes = ['i', 'me', 'my', 'new', 'anti', 'contra', 'counter', 'en', 'extra', 'hemi', 'hyper', 'in', 'non', 'over', 'pro', 'semi', 'syn', 'sub', 'ultra', 'under'];

const state = reactive({
  searchVal: '',
  results1: [{word: ''}],
  results2: [{word: ''}],
  suggestionsWithSuffixes: [''],
  suggestionsWithPrefixes: [''],
  suggestionsSynonyms: [''],
  suggestionsSynonymsWithSuffixes: [''],
  suggestionsSoundsLike: [''],
})

const generateNames = () => {
  state.suggestionsWithSuffixes = [];
  state.suggestionsWithPrefixes = [];
  state.suggestionsSynonyms = [];
  state.suggestionsSynonymsWithSuffixes = [];
  state.suggestionsSoundsLike = [];

  for (let i = 0; i < 10; i++) {
    const searchTerms = state.searchVal.split(' ');
    const randomWord = searchTerms[Math.floor(Math.random() * searchTerms.length)];
    const randomSuffix = suffixes[Math.floor(Math.random() * suffixes.length)];
    const randomName = `${randomWord}${randomSuffix}`;
    state.suggestionsWithSuffixes.push(randomName);
  }

  for (let i = 0; i < 10; i++) {
    const searchTerms = state.searchVal.split(' ');
    const randomWord = searchTerms[Math.floor(Math.random() * searchTerms.length)];
    const randomPrefix = prefixes[Math.floor(Math.random() * prefixes.length)];
    const randomName = `${randomPrefix}${randomWord}`;
    state.suggestionsWithPrefixes.push(randomName);
  }

  for (let i = 0; i < 10; i++) {
    const number = Math.floor(Math.random() * state.results1.length);
    const randomPrefix = state.results1[number].word;
    const randomName = `${randomPrefix}`;
    state.suggestionsSynonyms.push(randomName);
  }

  for (let i = 0; i < 10; i++) {
    const randomPrefix = state.results1[Math.floor(Math.random() * state.results1.length)].word;
    const randomSuffix = suffixes[Math.floor(Math.random() * suffixes.length)];
    const randomName = `${randomPrefix}${randomSuffix}`;
    state.suggestionsSynonymsWithSuffixes.push(randomName);
  }

  for (let i = 0; i < 10; i++) {
    const randomPrefix = state.results2[Math.floor(Math.random() * state.results2.length)].word;
    const randomName = `${randomPrefix}`;
    state.suggestionsSoundsLike.push(randomName);
  }
}

const copyText = (event: {
  target: any; preventDefault: () => void;
}) => {
  event.preventDefault();
  navigator.clipboard.writeText(event.target.innerText);
}

const handleSearch = async (event: { preventDefault: () => void; }) => {
  event.preventDefault();
  state.results1 = [];
  state.results2 = [];
  let response;
  const searchTerms = state.searchVal.split(' ');
  if (searchTerms.length > 1) {
    for (const term of searchTerms) {
      response = await axios.get(`https://api.datamuse.com/words?ml=${term}`);
      state.results1.push(...response.data);
      response = await axios.get(`https://api.datamuse.com/words?sl=${term}`)
      state.results2.push(...response.data);
    }
    const searchTerm = state.searchVal.split(' ').join('+');
    response = await axios.get(`https://api.datamuse.com/words?ml=${searchTerm}`);
    state.results1.push(...response.data);
    generateNames();
  } else {
    response = await axios.get(`https://api.datamuse.com/words?ml=${state.searchVal}`);
    state.results1.push(...response.data);

    response = await axios.get(`https://api.datamuse.com/words?sl=${state.searchVal}`);
    state.results2.push(...response.data);
    generateNames();
  }

}

</script>

<template>
  <main class="flex flex-col items-center w-full mt-20">
    <div class="flex items-center justify-center w-10/12">
      <form class="w-10/12">
        <label for="default-search"
               class="mb-2 text-sm font-medium text-gray-900 sr-only dark:text-white">Search</label>
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
      <button @click="generateNames"
              class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm px-4 py-2 dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800">
        R
      </button>
    </div>
    <div class="w-full flex flex-col items-center justify-center m-10" v-if="state.results1.length > 1">
      <div class="m-10 max-w-4xl">
        <div
            class="flex flex-wrap rounded box-decoration-slice bg-gradient-to-r from-indigo-600 to-pink-500 text-white px-2">
          <h2 class="underline text-3xl">Search terms with suffixes</h2>
          <button v-for="suggestion in state.suggestionsWithSuffixes" class="hover:underline hover:cursor-pointer p-8"
                  @click="copyText">
            {{ suggestion }}
          </button>
        </div>
      </div>
      <div class="m-10 max-w-4xl">
        <div
            class="flex flex-wrap rounded box-decoration-slice bg-gradient-to-r from-indigo-600 to-pink-500 text-white px-2">
          <h2 class="underline text-3xl">Search terms with prefixes</h2>
          <button v-for="suggestion in state.suggestionsWithPrefixes" class="hover:underline hover:cursor-pointer p-8"
                  @click="copyText">
            {{ suggestion }}
          </button>
        </div>
      </div>
      <div class="m-10 max-w-4xl">
        <div
            class="flex flex-wrap rounded box-decoration-slice bg-gradient-to-r from-indigo-600 to-pink-500 text-white px-2">
          <h2 class="underline text-3xl">Synonyms</h2>
          <button v-for="suggestion in state.suggestionsSynonyms" class="hover:underline hover:cursor-pointer p-8"
                  @click="copyText">
            {{ suggestion }}
          </button>
        </div>
      </div>
      <div class="m-10 max-w-4xl">
        <div
            class="flex flex-wrap rounded box-decoration-slice bg-gradient-to-r from-indigo-600 to-pink-500 text-white px-2">
          <h2 class="underline text-3xl">Synonyms with suffixes</h2>
          <button v-for="suggestion in state.suggestionsSynonymsWithSuffixes"
                  class="hover:underline hover:cursor-pointer p-8"
                  @click="copyText">
            {{ suggestion }}
          </button>
        </div>
      </div>
      <div class="m-10 max-w-4xl">
        <div
            class="flex flex-wrap rounded box-decoration-slice bg-gradient-to-r from-indigo-600 to-pink-500 text-white px-2">
          <h2 class="underline">Sounds like</h2>
          <button v-for="suggestion in state.suggestionsSoundsLike" class="hover:underline hover:cursor-pointer p-8"
                  @click="copyText">
            {{ suggestion }}
          </button>
        </div>
      </div>
    </div>
  </main>
</template>
