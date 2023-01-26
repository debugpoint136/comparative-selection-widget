<script>
  import MultiSelectGenome from './MultiSelectGenome.svelte';
  import { selection } from '../stores/selection.js';

  export let sourceGenome;
  function handleSelection(event) {
    const { selected, selectedIds, unselected } = event.detail;
    let yFilter = unselected.map((itemY) => itemY.name);
    let xFilter = selected.map((itemX) => itemX.name);
    $selection = $selection.filter((d) => !yFilter.includes(d.name));
    $selection = $selection.filter((d) => !xFilter.includes(d.name));
    $selection = [...$selection, ...selected];
  }
</script>

{#if sourceGenome}
  <div class="lg:w-1/2 w-full mb-6 lg:mb-0">
    <h1 class="sm:text-3xl text-2xl font-medium title-font mb-2 text-gray-600">
      Secondary genome :
    </h1>
    <small> > indicates the recommended assembly of that species</small>

    <div class="h-1 w-32 mb-2 bg-orange-500 mt-11 rounded" />
  </div>
  {#each sourceGenome.children as item}
    <div class="flex justify-between">
      <MultiSelectGenome
        name={item.name}
        children={item.children}
        on:select={handleSelection}
      />
    </div>
  {/each}
{:else}
  <!-- <p>make a selection</p> -->
{/if}
