<script lang="ts">
  import {
    linearPathFromAudioData,
    getNormalizeDataFromMaxMultipler,
    getFilterDataWithOffset,
  } from "waveform-path";

  export let peaks: Float32Array;
  export let width: number;
  export let maxMultiplier: number;
  export let zoom: number;
  export let start: number | undefined;
  export let end: number | undefined;
  $: console.log(start, end);
  let id = self.crypto.randomUUID();
  let path = "";
  $: peaks, width, calculatePath();

  function calculatePath() {
    if (!peaks?.length) return;
    console.time(`calculating path ${id}`);
    path = linearPathFromAudioData(
      getNormalizeDataFromMaxMultipler(
        getFilterDataWithOffset([peaks], width, start, end) as any,
        8
      ) as any,
      {
        samples: width,
        type: "steps",
        paths: [
          { d: "L", sx: 0, sy: 0, ex: 50, ey: 100 },
          { d: "L", sx: 50, sy: 100, ex: 100, ey: 0 },
        ],
        width: width,
        left: 0,
        top: 5,
        start,
        end,
      }
    );
    console.timeEnd(`calculating path ${id}`);
  }
</script>

<svg width="100%" height="100%">
  <line
    x1="0"
    y1="55"
    x2="100%"
    y2="55"
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
