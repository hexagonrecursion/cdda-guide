<script lang="ts">
import { getContext } from "svelte";
import type { CddaData } from "../../data";
import Recipe from "../Recipe.svelte";

export let item_id: string;

let data = getContext<CddaData>("data");

let recipes = data
  .byType("recipe")
  .filter((x) => x.result === item_id && !x.obsolete);
</script>

{#if recipes.length}
  {#each recipes as recipe (recipe)}
    <Recipe {recipe} />
  {/each}
{/if}
