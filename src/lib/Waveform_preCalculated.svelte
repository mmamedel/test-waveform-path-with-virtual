<script lang="ts">
  import { onMount } from "svelte";
  import { linearPathFromAudioData, getFilterData } from "waveform-path";

  export let peaks: number[] = [];
  export let width: number = 0;
  onMount(() => {
    console.log(peaks);
  });
  let path = "";
  $: peaks, width, calculatePath();

  function calculatePath() {
    if (!peaks?.length) return;
    console.log(width, peaks.length);
    path = linearPathFromAudioData([peaks], {
      samples: peaks.length,
      type: "steps",
      paths: [
        { d: "L", sx: 0, sy: 0, ex: 50, ey: 100 },
        { d: "L", sx: 50, sy: 100, ex: 100, ey: 0 },
      ],
      width: width,
      left: 0,
    });
  }
</script>

<svg width="100%" height="100%">
  <line
    x1="0"
    y1="50"
    x2="100%"
    y2="50"
    style="stroke-width: 0.5px; stroke: black"
  />
  <path
    style="
        fill: none;
        stroke-width: 0.5px;
        stroke-linecap: round;
        stroke: blue;
      "
    id="waveform"
    d={path}
  ></path>
</svg>
