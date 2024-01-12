<script lang="ts">
  import { createVirtualizer } from "@tanstack/svelte-virtual";
  import { onMount } from "svelte";
  import {
    getAudioData,
    getFilterData,
    getFramesData,
    getNormalizeData,
  } from "waveform-path";
  import Waveform from "./Waveform.svelte";

  let virtualItemEls: HTMLDivElement[] = [];
  export let scrollEl: HTMLDivElement | null;
  export let scrollElWidth: number;

  export let zoom = 6;

  $: virtualizer = createVirtualizer<HTMLDivElement, HTMLDivElement>({
    horizontal: true,
    count: zoom,
    getScrollElement: () => scrollEl,
    estimateSize: () => scrollElWidth,
    overscan: 1,
  });
  $: {
    if (virtualItemEls.length)
      virtualItemEls.forEach((el) => $virtualizer.measureElement(el));
  }

  let chunks: number[][] = new Array(zoom).fill([]);
  let frameData: Float32Array[];
  $: filteredData =
    frameData &&
    scrollElWidth &&
    zoom &&
    getFilterData(frameData, scrollElWidth * zoom);

  $: frameData, scrollElWidth, zoom, calculateChunks();

  function calculateChunks() {
    if (!filteredData) {
      chunks = [];
      return;
    }
    console.log("recalculating");
    const normalized = getNormalizeData(filteredData);
    const total = normalized[0].length;
    for (let i = 0; i < zoom; i++) {
      chunks[i] = normalized[0].slice(
        (total / zoom) * i,
        (total / zoom) * (i + 1)
      );
    }
  }

  onMount(async () => {
    const audioData = (await getAudioData(
      "https://d34ji3l0qn3w2t.cloudfront.net/9a76a2dd-4af8-44af-99a5-a8181b143596/1/osg_LSB_090_r720P.mp4"
    )) as AudioBuffer;
    frameData = getFramesData(audioData, 1, false, 0);
  });
</script>

<div
  class="list"
  style="position: relative; height: 500px; width: {$virtualizer.getTotalSize()}px;"
>
  {#each $virtualizer.getVirtualItems() as col, idx (col.index)}
    <div
      class:list-item-even={col.index % 2 === 0}
      class:list-item-odd={col.index % 2 === 1}
      style="position: absolute; top: 0; left: 0; width: {col.size}px; height: 100%; transform: translateX({col.start}px);"
      bind:this={virtualItemEls[idx]}
    >
      <Waveform peaks={chunks[col.index]} width={scrollElWidth} />
    </div>
  {/each}
</div>

<style>
  .list {
    background-color: lightgray;
  }
  .list-item-even {
    background-color: rgba(200, 200, 200, 0.4);
  }
</style>
