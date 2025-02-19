<script lang="ts">
import { data, mapType, singularName } from "./data";
import Fuse from "fuse.js";
import FurnitureSymbol from "./types/item/FurnitureSymbol.svelte";
import ItemSymbol from "./types/item/ItemSymbol.svelte";

const SEARCHABLE_TYPES = new Set([
  "item",
  "monster",
  "furniture",
  "vehicle_part",
  "tool_quality",
]);

let fuse: Fuse<any>;
$: fuse = new Fuse(
  [...($data?.all() ?? [])].filter(
    (x) => typeof x.id === "string" && SEARCHABLE_TYPES.has(mapType(x.type))
  ),
  {
    keys: ["id", "name"],
    getFn: (obj: any, path: string | string[]): string | string[] => {
      if (path[0] === "id") return obj.id ?? obj.abstract ?? "";
      if (path[0] === "name") return singularName(obj);
    },
    ignoreFieldNorm: true,
    ignoreLocation: true,
    minMatchCharLength: 2,
    threshold: 0.2,
  }
);

export let search: string;

function filter(text: string): Map<string, any[]> {
  const results = fuse.search(text, { limit: 100 });
  const byType = new Map<string, any[]>();
  for (const { item } of results) {
    const mappedType = mapType(item.type);
    if (!SEARCHABLE_TYPES.has(mappedType)) continue;
    if (!byType.has(mappedType)) byType.set(mappedType, []);
    byType.get(mappedType).push(item);
  }
  return byType;
}

$: matchingObjects = search && search.length > 1 && $data && filter(search);

$: history.replaceState({ search }, "");
</script>

{#if matchingObjects}
  {#each [...matchingObjects.keys()] as type}
    <h1>{type.replace(/_/g, " ")}</h1>
    <ul>
      {#each matchingObjects.get(type) as obj}
        <li>
          {#if type === "furniture"}
            <FurnitureSymbol item={$data._flatten(obj)} />
          {:else if type === "item" || type === "monster"}
            <ItemSymbol item={$data._flatten(obj)} />
          {/if}
          <a href="#/{mapType(obj.type)}/{obj.id}"
            >{singularName($data._flatten(obj))}</a>
        </li>
      {/each}
    </ul>
  {:else}
    <em>No results.</em>
  {/each}
{:else}
  <pre>...</pre>
{/if}
