<script>
  import { MultiSelect } from 'carbon-components-svelte';
  export let name;
  export let children;
  function getChildrenEntries() {
    let res = children.map((d, index) => {
      let tmp = d;
      d.id = d.name;
      d.text = d.querygenome;

      return tmp;
    });
    return res;
  }
</script>

{#if children}
  <MultiSelect
    type="inline"
    titleText={name}
    label="Select target genomes..."
    items={getChildrenEntries()}
    on:select
  />
{:else}
  <p>waiting...</p>
  {console.log(children)}
{/if}
