<template>
  <div style="margin: 2em">
    <div style="font-size: 2em">色計算機</div>
    <div
      v-for="c of colorsRGB"
      :style="{
        width: `${100 / colorsRGB.length}%`,
        display: 'inline-block',
        backgroundColor: `rgb(${c[0]}, ${c[1]}, ${c[2]})`,
        height: '1em',
      }"
    ></div>
    <div>モード</div>
    <select v-model="type">
      <option value="hsl">HSL</option>
      <option value="hsv">HSV</option>
      <option value="rgb">RGB</option></select
    ><select v-if="type !== 'rgb'" v-model="near">
      <option :value="true">時計回り</option>
      <option :value="false">反時計回り</option>
    </select>
    <div>開始色</div>
    <template v-if="type === 'rgb'">
      <div>
        <input
          type="range"
          min="0"
          max="255"
          step="1"
          v-model.number="start[0]"
        />
      </div>
      <div>
        <input
          type="range"
          min="0"
          max="255"
          step="1"
          v-model.number="start[1]"
        />
      </div>
      <div>
        <input
          type="range"
          min="0"
          max="255"
          step="1"
          v-model.number="start[2]"
        /></div
    ></template>
    <template v-else>
      <input
        type="range"
        min="0"
        max="360"
        step="1"
        v-model.number="start[0]"
        style="display: block"
      />
      <input
        type="range"
        min="0"
        max="100"
        step="1"
        v-model.number="start[1]"
        style="display: block"
      />
      <input
        type="range"
        min="0"
        max="100"
        step="1"
        v-model.number="start[2]"
        style="display: block"
      />
    </template>
    <div>終了色</div>
    <template v-if="type === 'rgb'">
      <div>
        <input
          type="range"
          min="0"
          max="255"
          step="1"
          v-model.number="end[0]"
        />
      </div>
      <div>
        <input
          type="range"
          min="0"
          max="255"
          step="1"
          v-model.number="end[1]"
        />
      </div>
      <div>
        <input
          type="range"
          min="0"
          max="255"
          step="1"
          v-model.number="end[2]"
        /></div
    ></template>
    <template v-else>
      <input
        type="range"
        min="0"
        max="360"
        step="1"
        v-model.number="end[0]"
        style="display: block"
      />
      <input
        type="range"
        min="0"
        max="100"
        step="1"
        v-model.number="end[1]"
        style="display: block"
      />
      <input
        type="range"
        min="0"
        max="100"
        step="1"
        v-model.number="end[2]"
        style="display: block"
      />
    </template>
    <div>
      <div>横幅</div>
      <input type="number" v-model.number="steps" />
    </div>
    <div>
      <div>縦幅</div>
      <input type="number" v-model.number="numGlasses" />
    </div>
    <div>
      <div>計算結果(横向き)</div>
      <div>下ー＞上</div>
      <div
        v-for="result of results"
        style="
          text-align: center;
          overflow-x: auto;
          height: 2em;
          width: max-content;
        "
      >
        <div
          v-for="r of result"
          style="display: inline-block; width: 2em; height: 2em"
          :style="{
            backgroundColor: `rgb(${colorNums[r][0]}, ${colorNums[r][1]}, ${colorNums[r][2]})`,
          }"
        >
          {{ colorNames[r] }}
        </div>
      </div>
    </div>
    <div>
      <div>必要数</div>
      <div v-for="(_, i) of indegrants">
        <div
          :style="{
            backgroundColor: `rgb(${colorNums[i][0]}, ${colorNums[i][1]}, ${colorNums[i][2]})`,
          }"
          style="height: 1em; width: 1em; display: inline-block"
        ></div>
        <span>{{ colorNames[i] }}: {{ indegrants[i] }}</span>
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
  const steps = ref(10);
  const type = ref("hsv");
  const start = ref([0, 100, 100]);
  const end = ref([360, 100, 100]);
  const near = ref(true);
  const numGlasses = ref(8);
  const colors = computed(() => {
    const ret: number[][] = [];
    switch (type.value) {
      case "hsl":
      case "hsv":
        for (let i = 0; i <= steps.value; i++) {
          const t = i / steps.value;
          const h = near.value
            ? start.value[0] + (end.value[0] - start.value[0]) * t
            : (start.value[0] + -(end.value[0] + start.value[0]) * t + 360) %
              360;
          const s = start.value[1] + (end.value[1] - start.value[1]) * t;
          const l = start.value[2] + (end.value[2] - start.value[2]) * t;
          ret.push([h, s, l]);
        }
        break;

      case "rgb":
        for (let i = 0; i <= steps.value; i++) {
          const t = i / steps.value;
          const r = start.value[0] + (end.value[0] - start.value[0]) * t;
          const g = start.value[1] + (end.value[1] - start.value[1]) * t;
          const b = start.value[2] + (end.value[2] - start.value[2]) * t;
          ret.push([r, g, b]);
        }
        break;
      default:
        break;
    }
    return ret;
  });
  const colorsRGB = computed(() => {
    return colors.value.map((c) => {
      switch (type.value) {
        case "hsl":
          return hslToRgb(c[0], c[1], c[2]);
        case "hsv":
          return hsvToRgb(c[0], c[1], c[2]);
        case "rgb":
          return c;
        default:
          return [0, 0, 0];
      }
    });
  });

  function hslToRgb(h: number, s: number, l: number): number[] {
    s /= 100;
    l /= 100;
    const k = (n: number) => (n + h / 30) % 12;
    const a = s * Math.min(l, 1 - l);
    const f = (n: number) => l - a * Math.max(-1, Math.min(1, k(n) - 3)) * 1;
    const r = Math.round(f(0) * 255);
    const g = Math.round(f(8) * 255);
    const b = Math.round(f(4) * 255);
    return [r, g, b];
  }

  function hsvToRgb(h: number, s: number, v: number): number[] {
    s /= 100;
    v /= 100;
    const i = Math.floor(h / 60);
    const f = h / 60 - i;
    const p = v * (1 - s);
    const q = v * (1 - f * s);
    const t = v * (1 - (1 - f) * s);
    let r: number, g: number, b: number;
    switch (i % 6) {
      case 0:
        r = v;
        g = t;
        b = p;
        break;
      case 1:
        r = q;
        g = v;
        b = p;
        break;
      case 2:
        r = p;
        g = v;
        b = t;
        break;
      case 3:
        r = p;
        g = q;
        b = v;
        break;
      case 4:
        r = t;
        g = p;
        b = v;
        break;
      case 5:
        r = v;
        g = p;
        b = q;
        break;
      default:
        r = 0;
        g = 0;
        b = 0;
    }
    return [Math.round(r * 255), Math.round(g * 255), Math.round(b * 255)];
  }
  const colorHexes = [
    "F9FFFE",
    "9D9D97",
    "474F52",
    "1D1D21",
    "835432",
    "B02E26",
    "F9801D",
    "FED83D",
    "80C71F",
    "5E7C16",
    "169C9C",
    "3AB3DA",
    "3C44AA",
    "8932B8",
    "C74EBD",
    "F38BAA",
  ];
  const colorNames = [
    "白",
    "薄灰",
    "灰",
    "黒",
    "茶",
    "赤",
    "橙",
    "黄",
    "黄緑",
    "緑",
    "空",
    "水",
    "青",
    "紫",
    "赤紫",
    "ピン",
  ];
  const colorNums: number[][] = [];
  for (const h of colorHexes) {
    colorNums.push([
      parseInt(h.substring(0, 2), 16),
      parseInt(h.substring(2, 4), 16),
      parseInt(h.substring(4), 16),
    ]);
  }
  const results = computed(() => {
    const ret: number[][] = [];
    for (const color of colorsRGB.value) {
      const rr: number[] = [];
      ret.push(rr);
      const currentColor = [0, 0, 0];
      let intensity = 0.5;
      for (let i = 0; i < numGlasses.value; i++) {
        let min = Infinity;
        let minIndex = 0;
        for (let j = 0; j < colorNums.length; j++) {
          const distance = Math.sqrt(
            Math.pow(
              color[0] - (currentColor[0] + colorNums[j][0] * intensity),
              2
            ) +
              Math.pow(
                color[1] - (currentColor[1] + colorNums[j][1] * intensity),
                2
              ) +
              Math.pow(
                color[2] - (currentColor[2] + colorNums[j][2] * intensity),
                2
              )
          );
          if (distance < min) {
            min = distance;
            minIndex = j;
          }
        }
        currentColor[0] += colorNums[minIndex][0] * intensity;
        currentColor[1] += colorNums[minIndex][1] * intensity;
        currentColor[2] += colorNums[minIndex][2] * intensity;
        rr.push(minIndex);
        intensity *= 0.5;
      }
      rr.reverse();
      const f = rr[0]
      while (f === rr[0]) {
        rr.shift();
      }
      rr.unshift(f)
    }
    return ret;
  });
  const indegrants = computed(() => {
    const ret = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
    for (const c of results.value.flat()) {
      ret[c]++;
    }
    return ret;
  });
</script>
