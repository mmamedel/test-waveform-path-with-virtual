<script lang="ts">
  import { createVirtualizer } from "@tanstack/svelte-virtual";
  import { onMount } from "svelte";
  import { getAudioData, getFramesData, getMaxMultipler } from "waveform-path";
  import Waveform from "./Waveform.svelte";

  let virtualItemEls: HTMLDivElement[] = [];
  export let scrollEl: HTMLDivElement | null;
  export let scrollElWidth: number;
  export let zoom = 6;
  let audioEl: HTMLAudioElement | undefined;

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

  let frameData: Float32Array[] = [[]];
  $: maxMultiplier = frameData ? getMaxMultipler(frameData) : 0;
  $: console.log(maxMultiplier);
  $: frameData && console.log(frameData[0].length);

  onMount(async () => {
    const audioData = (await getAudioData(
      // "https://d34ji3l0qn3w2t.cloudfront.net/432de493-2fcc-4e1f-a9b1-8c66a939122c/1/jwb-108_LSB_08_r720P.mp4"
      "https://d34ji3l0qn3w2t.cloudfront.net/9a76a2dd-4af8-44af-99a5-a8181b143596/1/osg_LSB_090_r720P.mp4"
    )) as AudioBuffer;
    // const audioCtx = new AudioContext();
    if (audioEl) {
      // const audioSourceNode = audioCtx.createMediaElementSource(audioEl);
      // const analyserNode = audioCtx.createAnalyser();
      // console.log({ frequencyBinCount: analyserNode.frequencyBinCount });
      // const bufferOutput = audioCtx.createBuffer(audioSourceNode.channelCount, analyserNode.frequencyBinCount, audioCtx.sampleRate)
      // audioSourceNode.connect(analyserNode);
      // analyserNode.connect(audioCtx.destination);
      // audioCtx.decodeAudioData()
      frameData = getFramesData(audioData, 1, false, 0);
    }
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
      <Waveform
        peaks={frameData[0]}
        width={scrollElWidth}
        {maxMultiplier}
        {zoom}
        start={(frameData[0].length / zoom) * col.index}
        end={(frameData[0].length / zoom) * (col.index + 1)}
      />
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
