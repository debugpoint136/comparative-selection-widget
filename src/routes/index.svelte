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
  let UUID;

  const POST_DATAHUB_URL =
    'https://hcwxisape8.execute-api.us-east-1.amazonaws.com/dev/datahub';
  let uploaded = false;
  let savingnotification = false;
  let DATAHUB_URL;

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

  async function saveinmongo() {
    UUID = uuid.v4();
    uploaded = false;
    savingnotification = true;

    console.log('ok will save in mongo');
    const toPost = {
      _id: UUID,
      files: [],
      hub: { content: selection },
      comments: 'TEST',
      compositegraphdata: {},
      registered: Date(),
      username: 'dpuru',
    };

    console.log('Posting request to mongo API...');

    try {
      const response = await axios.post(POST_DATAHUB_URL, toPost);
      console.log(response);
      let resBody = response.data.body;
      if (resBody.hasOwnProperty('id')) {
        uploaded = true;
        savingnotification = false;
        DATAHUB_URL = `https://epigenomegateway.wustl.edu/browser/?genome=hg19&hub=${POST_DATAHUB_URL}/${UUID}`;
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
      on:click={() => saveinmongo()}
      class="flex mx-auto my-16 text-white bg-indigo-500 border-0 py-2 px-8 focus:outline-none hover:bg-indigo-600 rounded text-lg"
      >Save Datahub</button
    >

    {#if savingnotification}
      <p>Saving...please wait..</p>
    {:else if DATAHUB_URL !== undefined}
      <a
        href={DATAHUB_URL}
        target="_blank"
        class="flex justify-center max-w-xl mx-auto my-16 text-white bg-pink-500 border-0 py-2 px-8 focus:outline-none hover:bg-pink-600 rounded text-lg"
      >
        Open WashU Epigenome Browser
      </a>
    {/if}
  </div>
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
