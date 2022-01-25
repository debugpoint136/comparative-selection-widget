<script>
  import {
    Accordion,
    AccordionItem,
    TileGroup,
    RadioTile,
  } from 'carbon-components-svelte';
  import { treeData } from '$lib/data';
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  let sourceGenome = '';
  let sourceOrganism = '';

  function updateValue(value, organism, index) {
    sourceGenome = value;
    sourceOrganism = organism;
    let sourceObject = treeData.children[index].children.filter(
      (d) => d.name === value
    );
    dispatch('sourceSelect', sourceObject);
  }
</script>

<div class="lg:w-1/2 w-full mb-6 lg:mb-0">
  <h1 class="sm:text-3xl text-2xl font-medium title-font mb-2 text-gray-600">
    Source genome :
  </h1>
  <h1 class="sm:text-3xl text-2xl font-bold title-font mb-2 text-gray-900">
    {sourceGenome}
  </h1>
  <div class="h-1 w-32 bg-indigo-500 rounded" />
</div>
{#key sourceGenome}
  <Accordion align="start">
    {#each treeData.children as item, index}
      <AccordionItem
        open={sourceOrganism === item.name ? true : false}
        disabled={!item.children.length}
        title={item.name}
      >
        <TileGroup legend="Pick Source genome">
          {#each item.children as source}
            <RadioTile
              on:change={() => updateValue(source.name, item.name, index)}
              disabled={!source.children.length}
              value={source.name}
              checked={source.name === sourceGenome ? true : false}
              >{source.name}
            </RadioTile>
          {/each}
        </TileGroup>
      </AccordionItem>
    {/each}
  </Accordion>
{/key}
