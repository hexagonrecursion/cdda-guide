<script lang="ts">
import { getContext } from "svelte";
import { CddaData, singularName } from "../data";
import type { AmmunitionType } from "../types";
import ThingLink from "./ThingLink.svelte";

export let item: AmmunitionType;

const data = getContext<CddaData>("data");

const compatible = data.byType("item").flatMap((x) => {
  if (x.type !== "AMMO" || !x.id) return [];
  if (x.ammo_type === item.id) return [x];
  return [];
});
compatible.sort((a, b) => singularName(a).localeCompare(singularName(b)));
</script>

<h1>Ammunition Type: {singularName(item)}</h1>
<section>
  <h1>Compatible Variants</h1>
  <ul>
    {#each compatible as ammo}
      <li>
        <ThingLink type="item" id={ammo.id} />
        {#if ammo.id === item.default}(default){/if}
      </li>
    {/each}
  </ul>
</section>
