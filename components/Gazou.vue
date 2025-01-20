<template>
  <div>
    <canvas
      ref="canvasRef"
      style="width: 25%; image-rendering: pixelated"
    ></canvas
    >==><canvas
      ref="resultRef"
      style="width: 25%; image-rendering: pixelated"
    ></canvas>

    <div>
      <div>横幅</div>
      <input type="number" v-model.number="width" />
    </div>
    <div>
      <div>縦幅</div>
      <input type="number" v-model.number="height" />
    </div>
    <div>
      <div>最初に色を作る高さ</div>
      <input type="number" v-model.number="numGlasses" />
    </div>
    <div>
      <input type="file" @change="onSelect" />
    </div>
  </div>
  <div>
    <div>計算結果(横向き、「/」は無色のガラス)</div>
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
          backgroundColor: `rgb(${colorNumsDisp[r][0]}, ${colorNumsDisp[r][1]}, ${colorNumsDisp[r][2]})`,
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
    <span>無色: {{ indegrants[-1] }}</span>
  </div>
</template>
<script setup lang="ts">
  const canvasRef = ref<HTMLCanvasElement>();
  const resultRef = ref<HTMLCanvasElement>();
  const width = ref(16);
  const height = ref(16);
  const imageData = ref<number[]>([]);
  const imageColors = ref<number[][][]>([]);
  onMounted(() => {
    if (!canvasRef.value || !resultRef.value) {
      return;
    }
    const cv = canvasRef.value;
    const rv = resultRef.value;
    watchEffect(() => ((cv.width = width.value), (rv.width = width.value)));
    watchEffect(
      () => (
        (cv.height = height.value + numGlasses.value + 1),
        (rv.height = height.value + numGlasses.value + 1)
      )
    );
    watch(imageData, (d) => {
      if (!canvasRef.value) {
        return;
      }
      const ctx = canvasRef.value.getContext("2d");
      if (!ctx) {
        return;
      }
      const get = (x: number, y: number) => {
        return [
          d[y * width.value * 4 + x * 4 + 0],
          d[y * width.value * 4 + x * 4 + 1],
          d[y * width.value * 4 + x * 4 + 2],
        ];
      };
      const low: number[][] = [];
      for (let i = 0; i < width.value; i++) {
        low.push(get(i, height.value - 1));
      }
      colorsRGB.value = low;
      imageColors.value.length = 0;
      for (let i = 0; i < width.value; i++) {
        const c: number[][] = [];
        imageColors.value.push(c);
        for (let j = 0; j < height.value; j++) {
          c.push(get(i, height.value - j - 1));
        }
      }
    });
    watch(final, (f) => {
      if (!resultRef.value) {
        return;
      }
      const ctx = resultRef.value.getContext("2d");
      if (!ctx) {
        return;
      }
      ctx.clearRect(0, 0, resultRef.value.width, resultRef.value.height);
      for (let i = 0; i < width.value; i++) {
        for (let j = 0; j < height.value + numGlasses.value + 1; j++) {
          ctx.fillStyle = `rgb(${f[i][j][0]}, ${f[i][j][1]}, ${f[i][j][2]})`;
          ctx.fillRect(i, height.value - j + numGlasses.value, 1, 1);
        }
      }
    });
  });
  const onSelect = (e: Event) => {
    const file = (e.target as HTMLInputElement).files?.[0];
    if (!file) {
      return;
    }
    const image = new Image();
    image.onload = () => {
      canvasRef.value
        ?.getContext("2d")
        ?.drawImage(
          image,
          0,
          0,
          image.width,
          image.height,
          0,
          0,
          width.value,
          height.value
        );
      URL.revokeObjectURL(image.src);
      imageData.value = Array.from(
        canvasRef.value
          ?.getContext("2d")
          ?.getImageData(0, 0, width.value, height.value).data!
      );
    };
    image.src = URL.createObjectURL(file);
  };

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
    "青緑",
    "空",
    "青",
    "紫",
    "赤紫",
    "桃",
  ];
  colorNames[-1] = "/";
  const colorNums: number[][] = [];
  for (const h of colorHexes) {
    colorNums.push([
      parseInt(h.substring(0, 2), 16),
      parseInt(h.substring(2, 4), 16),
      parseInt(h.substring(4), 16),
    ]);
  }
  const colorNumsDisp = colorNums.slice();
  colorNumsDisp[-1] = [128, 255, 255];
  const numGlasses = ref(8);
  const colorsRGB = ref<number[][]>([]);
  const results = computed(() => {
    const ret: number[][] = [];
    let i = -1;
    for (const color of colorsRGB.value) {
      i++;
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
      const f = rr[0];
      while (f === rr[0]) {
        rr.shift();
        rr.push(-1);
      }
      rr.unshift(f);
      const currentColor2 = [
        colorNums[rr[0]][0],
        colorNums[rr[0]][1],
        colorNums[rr[0]][2],
      ];
      for (let j = 1; j < numGlasses.value; j++) {
        if (rr[j] === -1) {
          break;
        }
        currentColor2[0] = (currentColor2[0] + colorNums[rr[j]][0]) / 2;
        currentColor2[1] = (currentColor2[1] + colorNums[rr[j]][1]) / 2;
        currentColor2[2] = (currentColor2[2] + colorNums[rr[j]][2]) / 2;
      }
      for (let j = 0; j < height.value; j++) {
        const color = imageColors.value[i][j];
        let abs = Math.sqrt(
          (color[0] - currentColor2[0]) ** 2 +
            (color[1] - currentColor2[1]) ** 2 +
            (color[2] - currentColor2[2]) ** 2
        );
        let minIndex = -1;
        for (let k = 0; k < colorNums.length; k++) {
          const distance = Math.sqrt(
            (color[0] - (currentColor2[0] + colorNums[k][0]) / 2) ** 2 +
              (color[1] - (currentColor2[1] + colorNums[k][1]) / 2) ** 2 +
              (color[2] - (currentColor2[2] + colorNums[k][2]) / 2) ** 2
          );
          if (distance < abs) {
            abs = distance;
            minIndex = k;
          }
        }
        rr.push(minIndex);
        if (minIndex !== -1) {
          currentColor2[0] = (currentColor2[0] + colorNums[minIndex][0]) / 2;
          currentColor2[1] = (currentColor2[1] + colorNums[minIndex][1]) / 2;
          currentColor2[2] = (currentColor2[2] + colorNums[minIndex][2]) / 2;
        }
      }
    }
    return ret;
  });
  const indegrants = computed(() => {
    const ret = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
    ret[-1] = 0;
    for (const c of results.value.flat()) {
      ret[c]++;
    }
    return ret;
  });
  const final = computed(() => {
    const ret: number[][][] = [];
    for (const r of results.value) {
      const rr: number[][] = [];
      ret.push(rr);
      const currentColor = [0, 0, 0];
      for (const c of r) {
        if (c !== -1) {
          currentColor[0] = (colorNums[c][0] + currentColor[0]) / 2;
          currentColor[1] = (colorNums[c][1] + currentColor[1]) / 2;
          currentColor[2] = (colorNums[c][2] + currentColor[2]) / 2;
        }
        rr.push(currentColor.slice());
      }
    }
    console.log(ret);
    return ret;
  });
</script>
