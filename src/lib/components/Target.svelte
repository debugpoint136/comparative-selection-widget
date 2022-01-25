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
