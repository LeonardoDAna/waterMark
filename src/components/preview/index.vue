<template>
  <!-- <div></div> -->
  <canvas ref="canvasTestRef" width="200" height="200" class="preview_container"></canvas>
</template>

<script setup>
import { ref, watch, nextTick } from "vue";
const canvasTestRef = ref(null);

const props = defineProps({
  config: {
    type: Object,
    default: () => ({
      content: "测试水印",
      color: "#000000",
      fontSize: 16,
      gap: 10,
      rotate: 45,
      waterMark_width: 200,
      waterMark_height: 100,
    }),
  },
});

let clearRect = () => {
  const canvas = canvasTestRef.value;
  const ctx = canvas.getContext("2d");
  ctx.clearRect(0, 0, canvas.width, canvas.height);
};
let addWaterMark = () => {
  let config = {
    fontSize: props.config.fontSize,
    text: props.config.content,
    color: props.config.color,
    gap: props.config.gap,
    type: props.config.type,
    waterMark_width: props.config.waterMark_width,
    waterMark_height: props.config.waterMark_height,
  };

  const canvas = canvasTestRef.value;
  const ctx = canvas.getContext("2d");

  //   canvas.width = canvas.w;
  //   canvas.height = img.height * scale;

  ctx.font = `${config.fontSize}px Arial`;
  ctx.fillStyle = config.color; // 黑色文字，透明度为0.5
  ctx.textBaseline = "middle";

  let count = 10;
  let line = 0;
  let column = 0;

  let metrics = ctx.measureText(config.text);
  var textWidth = metrics.width;
  var textHeight = metrics.emHeightAscent + metrics.emHeightDescent;
  let block_width = textWidth + config.gap * 2;
  let block_height =
    textHeight >= config.waterMark_height ? textHeight : config.waterMark_height;

  let compute_column = canvas.width % block_width;
  let compute_line = canvas.height % block_height;

  count = compute_column * compute_line + 4;

  for (let index = 0; index < count; index++) {
    let x, y;

    // if (textWidth >= config.waterMark_width) {
    //   x = block_width * column - textWidth / 2;
    //   y = block_height * line;
    // } else {
    x = config.waterMark_width * column;
    y = block_height * line;
    // }

    column++;

    if (x + textWidth / 2 > canvas.width) {
      line++;
      column = 0;
    }

    // 将文字绘制到Canvas上
    ctx.save(); // 保存当前状态
    ctx.translate(x, y); // 移动到指定位置
    switch (config.type) {
      case 1:
        ctx.rotate((45 * Math.PI) / 180); // 旋转45度
        break;
      case 2:
        ctx.rotate((-45 * Math.PI) / 180); // 旋转45度
      case 3:
        break;
    }
    ctx.fillText(config.text, 0, 0); // 绘制文字
    ctx.restore(); // 恢复到之前的状态
  }
};
watch(
  () => props.config,
  async () => {
    await nextTick();
    clearRect();
    addWaterMark();
  },
  { deep: true, immediate: true }
);
</script>

<style scoped>
.preview_container {
  /* width: 500px;
  height: 200px; */
}
</style>
