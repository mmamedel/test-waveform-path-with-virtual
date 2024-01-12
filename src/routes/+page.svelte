<script lang="ts">
  import ColumnVirtualizerFixed from "$lib/ColumnVirtualizerFixed.svelte";
  let scrollEl: HTMLDivElement | null;
  let scrollElWidth: number;
  let zoom = 1;
</script>

<div
  class="scroll-container"
  bind:this={scrollEl}
  bind:clientWidth={scrollElWidth}
>
  <ColumnVirtualizerFixed {scrollEl} {zoom} {scrollElWidth} />
</div>
<input type="Range" min={1} max={1000} step="100" bind:value={zoom} />

<style>
  .scroll-container {
    height: 500px;
    width: 100vw;
    overflow-x: scroll;
    overflow-y: hidden;
  }
</style>
