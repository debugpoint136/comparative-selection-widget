<script>
  import '../app.css';
  import { onMount } from 'svelte';
  import { treeData } from '$lib/data';
  import { treeGenerator } from '$lib/hierarchy';
  import * as d3 from 'd3';
  import * as uuid from 'uuid';
  import axios from 'axios';

  const dispatcher = d3.dispatch('toggleCompareInput');
  $: selection = [];
  let elem;
  $: mongoSavedLinks = [];

  const POST_DATAHUB_URL =
    'https://hcwxisape8.execute-api.us-east-1.amazonaws.com/dev/datahub';
  let uploaded = false;
  let savingnotification = false;
  // let DATAHUB_URL;

  dispatcher.on('toggleCompareInput', (input) => {
    selection = input.map((d) => {
      let tmp = d;
      tmp.isSelected = true;
      tmp.showOnHubLoad = true;
      return tmp;
    });

    console.log(selection);
  });

  onMount(() => {
    treeGenerator(elem, treeData, dispatcher);
  });

  async function saveDatahubs() {
    mongoSavedLinks = []; // reset
    const selectionGroups = d3.group(selection, (d) => d.source);
    console.log(selectionGroups);
    selectionGroups.forEach(async (element) => {
      await saveinmongo(element);
    });
  }

  async function saveinmongo(selectionForGenome) {
    let UUID = uuid.v4();
    uploaded = false;
    savingnotification = true;
    const sourceGenome = selectionForGenome[0].source;

    console.log('ok will save in mongo');

    const toPost = {
      _id: UUID,
      files: [],
      hub: { content: selectionForGenome },
      comments: 'TEST',
      compositegraphdata: {},
      registered: Date(),
      username: 'dpuru',
    };

    console.log('Posting request to mongo API...');

    try {
      const response = await axios.post(POST_DATAHUB_URL, toPost);
      let resBody = response.data.body;
      let DATAHUB_URL;
      if (resBody.hasOwnProperty('id')) {
        uploaded = true;
        savingnotification = false;
        DATAHUB_URL = `https://epigenomegateway.wustl.edu/browser/?genome=${sourceGenome}&hub=${POST_DATAHUB_URL}/${UUID}`;
        mongoSavedLinks = mongoSavedLinks.concat({
          source: sourceGenome,
          url: DATAHUB_URL,
        });
        console.log('Created datahub:', DATAHUB_URL);
      }
    } catch (error) {
      throw new Error(error);
    }
  }
</script>

<main>
  <div class="max-w-4xl mx-auto px-8 py-2 text-green-500">
    <h4>Make selection</h4>
  </div>
  <div class="flex">
    <button
      on:click={() => saveDatahubs()}
      class="flex mx-auto my-16 text-white bg-indigo-500 border-0 py-2 px-8 focus:outline-none hover:bg-indigo-600 rounded text-lg"
      >Save Datahub</button
    >
  </div>
  {#if savingnotification}
    <p>Saving...please wait..</p>
  {:else if mongoSavedLinks.length}
    <span
      class="mx-2 inline-block py-1 px-2 rounded bg-indigo-50 text-indigo-500 text-xs font-medium tracking-widest"
      >OPEN IN WASH U EPIGENOME BROWSER</span
    >
    {#each mongoSavedLinks as item}
      <a href={item.url} target="_blank" class="">
        <div class="p-2 sm:w-1/2 w-full">
          <div class="bg-gray-100 rounded flex p-4 h-full items-center">
            <svg
              fill="none"
              stroke="currentColor"
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="3"
              class="text-indigo-500 w-6 h-6 flex-shrink-0 mr-4"
              viewBox="0 0 24 24"
            >
              <path d="M22 11.08V12a10 10 0 11-5.93-9.14" />
              <path d="M22 4L12 14.01l-3-3" />
            </svg>
            <span class="title-font font-medium">Source: {item.source}</span>
          </div>
        </div>
      </a>
    {/each}
  {/if}

  <div class="max-w-4xl mx-auto py-2" id="treePlaceholder" bind:this={elem} />
</main>

<style>
  :global(.node circle) {
    fill: #fff;
    stroke: steelblue;
    stroke-width: 3px;
  }

  :global(.node text) {
    font: 12px;
    @apply font-medium;
  }

  :global(.link) {
    fill: none;
    stroke: #ccc;
    stroke-width: 2px;
  }
</style>
