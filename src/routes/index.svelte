<script>
  import Target from '$lib/components/Target.svelte';
  import SourceOrganisms from '$lib/components/Source.svelte';
  import { selection } from '$lib/stores/selection';
  import '../all.css';
  import '../app.css';
  import * as uuid from 'uuid';
  import axios from 'axios';
  let sourceGenome;
  $: mongoSavedLinks = [];
  const POST_DATAHUB_URL =
    'https://hcwxisape8.execute-api.us-east-1.amazonaws.com/dev/datahub';
  let uploaded = false;
  let savingnotification = false;

  function getTimestamp() {
    const pad = (n, s = 2) => `${new Array(s).fill(0)}${n}`.slice(-s);
    const d = new Date();

    return `${pad(d.getFullYear(), 4)}-${pad(d.getMonth() + 1)}-${pad(
      d.getDate()
    )} ${pad(d.getHours())}:${pad(d.getMinutes())}:${pad(d.getSeconds())}.${pad(
      d.getMilliseconds(),
      3
    )}`;
  }

  function handleSource(event) {
    sourceGenome = event.detail[0];
    if ($selection.length > 0) {
      $selection = [];
    }
  }
  async function saveinmongo() {
    let UUID = uuid.v4();
    uploaded = false;
    savingnotification = true;

    console.log('ok will save in mongo');

    let selectedList = [...$selection];
    selectedList = selectedList.map((d) => {
      let tmp = d;
      tmp.isSelected = true;
      tmp.showOnHubLoad = true;
      return tmp;
    });

    const toPost = {
      _id: UUID,
      files: [],
      hub: { content: selectedList },
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
        DATAHUB_URL = `https://epigenomegateway.wustl.edu/browser/?genome=${sourceGenome.name}&hub=${POST_DATAHUB_URL}/${UUID}`;
        mongoSavedLinks = mongoSavedLinks.concat({
          source: sourceGenome.name,
          url: DATAHUB_URL,
          timestamp: getTimestamp(),
        });
        console.log('Created datahub:', DATAHUB_URL);
      }
    } catch (error) {
      throw new Error(error);
    }
  }
</script>

<section class="text-gray-600 body-font">
  <div class="container px-5 py-24 mx-auto">
    <div class="flex flex-wrap -m-4">
      <div class="p-4 md:w-1/3">
        <div
          class="h-full border-2 border-gray-200 border-opacity-60 rounded-lg overflow-hidden"
        >
          <SourceOrganisms on:sourceSelect={handleSource} />
        </div>
      </div>
      <div class="p-4 md:w-1/3">
        <div
          class="h-full border-2 border-gray-200 border-opacity-60 rounded-lg overflow-hidden"
        >
          {#if sourceGenome}
            {#key sourceGenome.name}
              <Target {sourceGenome} />
            {/key}
          {/if}
        </div>
      </div>
      <div class="p-4 md:w-1/3">
        <div
          class="h-full border-2 border-gray-200 border-opacity-60 rounded-lg overflow-hidden"
        >
          {#if $selection.length > 0}
            <div class="flex">
              <button
                on:click={() => saveinmongo()}
                class="flex mx-auto my-16 text-white bg-indigo-500 border-0 py-2 px-8 focus:outline-none hover:bg-indigo-600 rounded text-lg"
                >Save selection</button
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
                  <div class="p-2 w-full">
                    <div
                      class="bg-gray-100 rounded flex p-4 h-full items-center"
                    >
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
                      <span class="title-font font-medium"
                        >Source: {item.source}</span
                      >
                    </div>
                    <span
                      class="text-gray-400 mr-3 inline-flex items-center lg:ml-auto md:ml-0 ml-auto leading-none text-sm pr-3 py-1 border-r-2 border-gray-200"
                    >
                      {item.timestamp}
                    </span>
                  </div>
                </a>
              {/each}
            {/if}
          {/if}
        </div>
      </div>
    </div>
  </div>
</section>
