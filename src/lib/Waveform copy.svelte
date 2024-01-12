<script lang="ts">
  import { onMount } from "svelte";
  import { linearPathFromAudioData, getFilterData } from "waveform-path";

  export let peaks: Float32Array = new Float32Array();
  export let width: number = 0;
  onMount(() => {
    console.log(peaks);
  });
  let path = "";
  $: peaks, width, calculatePath();

  function calculatePath() {
    if (!peaks?.length) return;
    console.log(width, peaks.length);
    path = linearPathFromAudioData(getFilterData([peaks], width / 8), {
      samples: width / 8,
      type: "mirror",
      paths: [{ d: "V", sy: 0, x: 0, ey: 2000 }],
      width: width + 3,
      top: 190,
      left: 4,
    });
  }
</script>

<svg width="100%" height="100%">
  <path
    style="
        fill: none;
        stroke-width: 4px;
        stroke-linecap: round;
        stroke: lightgray;
      "
    id="waveform"
    d={path}
  ></path>
</svg>
