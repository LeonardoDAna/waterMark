<script setup>
import HelloWorld from "./components/HelloWorld.vue";
import dayjs from "dayjs";
import { nextTick, reactive, ref } from "vue";

const fileInput = ref();
const canvasRef = ref();

const textContent = ref(dayjs().format("YYYY-MM-DD HH:mm:ss"));

const type = ["左上", "右上", "垂直", "水平"];

let typeIndex = ref(0);

const setupData = reactive({
  photoList: [],
});

const upload = async () => {
  fileInput.value.click();
};

const loadImageFromBase64 = (blobimage) => {
  return new Promise((resolve, reject) => {
    const blob = new Blob([blobimage], {
      type: blobimage.type || "image/jpg",
    }); //类型一定要写！！
    const reader = new FileReader();
    reader.readAsDataURL(blob);
    reader.onload = () => {
      const image = new Image();
      image.src = reader.result;
      image.onload = () => resolve(image);
      image.onerror = () => reject(new Error("Failed to load image from Base64"));
    };
    reader.onerror = (error) => reject(error);
  });
};

const getFiles = async (e) => {
  await nextTick();
  let requestList = [];
  for (let index = 0; index < e.target.files.length; index++) {
    const file = e.target.files[index];
    // setupData.photoList[index] = file;
    requestList.push(loadImageFromBase64(file));

    // let reader = new FileReader();
    // reader.readAsDataURL(file);
    // reader.onload = function (e) {
    // getImgInfo(index, e.target.result);
    // };
  }

  Promise.all(requestList)
    .then((images) => {
      setupData.photoList = images;
      console.log(images);
    })
    .catch((error) => {
      console.log(error);
    });
};

const getImgInfo = (index, imgUrl) => {
  let img = new Image();
  img.src = imgUrl;
  img.onload = function () {
    setupData.photoList[index].coverUrl = imgUrl;
    setupData.photoList[index].height = img.height;
    setupData.photoList[index].width = img.width;
    console.log("图片原始高度", img.height);
    console.log("图片原始宽度", img.width);
  };
};
const addWaterMark = () => {
  let img = setupData.photoList[0];
  let proportion = img.width / img.height;
  let scale = 0.4;

  let config = {
    fontSize: 16,
    text: "蟹黄包秘方持有者",
    gap: 300 * scale,

    waterMark_width: 100 * proportion * scale,
    waterMark_height: 500 * proportion * scale,
  };

  const canvas = canvasRef.value;
  const ctx = canvas.getContext("2d");

  canvas.width = img.width * scale;
  canvas.height = img.height * scale;

  ctx.drawImage(img, 0, 0, img.width * scale, img.height * scale);
  // 旋转 45 度让文字变倾斜

  const fontSize = Math.floor(canvas.width / 30);

  ctx.font = `${fontSize}px Arial`;
  ctx.fillStyle = "rgba(0, 0, 0, 0.3)"; // 黑色文字，透明度为0.5
  ctx.textBaseline = "middle";

  // 计算文字宽度以实现居中对齐
  // var textWidth = ctx.measureText(config.text).width;
  // var x = (canvas.width - textWidth) / config.gap;
  // var y = canvas.height / config.gap;

  let count = 100;

  // ctx.rotate((Math.PI / 180) * 45);
  let line = 0;
  let column = 0;

  let metrics = ctx.measureText(config.text);
  var textWidth = metrics.width;
  var textHeight = metrics.emHeightAscent + metrics.emHeightDescent;
  let block_width = textWidth + config.gap * 2;
  let block_height =
    textHeight >= config.waterMark_height ? textHeight : config.waterMark_height;

  ctx.translate(canvas.width / 2, canvas.height / 2);
  ctx.rotate((45 * Math.PI) / 180);
  ctx.translate(-canvas.width / 2, -canvas.height / 2);
  ctx.translate(-1000, -1000);

  for (let index = 0; index <= count; index++) {
    let x, y;

    if (textWidth >= config.waterMark_width) {
      // x = textWidth * column + config.gap;
      x = block_width * column;
      y = block_height * line + block_height / 2;
    } else {
      x = config.waterMark_width * column + config.gap;
      y = (config.waterMark_width + textWidth) / 2;
    }

    ctx.strokeStyle = "green";
    ctx.lineWidth = 30;
    // ctx.strokeRect(block_width * column, block_height * line, block_width, block_height);

    column++;

    if (textWidth * column >= canvas.width) {
      line++;
      column = 0;
    }

    // ctx.translate(canvas.width / 2, canvas.height / 2);
    // ctx.translate(x, y);
    // ctx.rotate((45 * Math.PI) / 180);

    // ctx.strokeRect(block_width * column, block_height * line, block_width, block_height);

    // ctx.fillText(config.text, 0, 0);

    // 将文字绘制到Canvas上
    ctx.fillText(config.text, x, y);
    // ctx.translate(-x, -y);

    // ctx.rotate((-45 * Math.PI) / 180);
  }
};

const downloadImages = () => {
  let canvas = canvasRef.value;
  const base64Img = canvas.toDataURL("image/png", 0.1);
  var a = document.createElement("a"); // 生成一个a元素
  var event = new MouseEvent("click"); // 创建一个单击事件
  a.download = dayjs().valueOf("YYYY-MM-DD HH:mm:ss"); // 设置图片名称
  a.href = base64Img; // 将生成的URL设置为a.href属性
  a.dispatchEvent(event);
};
</script>

<template>
  <button @click="upload">批量上传</button>
  <input
    ref="fileInput"
    type="file"
    id="fileInput"
    name="file"
    multiple
    @change="getFiles"
  />

  <button>单个上传</button>
  <div>
    <h4>图片列表</h4>
    <div class="imgList">
      <template v-for="(item, index) in setupData.photoList" :key="index">
        <div class="imgItem">
          <img :src="item.coverUrl" alt="" />
          <div class="imgItem-info">
            <div>{{ item.name }}</div>
            <div>{{ item.size }}</div>
          </div>
        </div>
      </template>
    </div>
  </div>
  <div>操作栏</div>
  <div>取消</div>
  <div @click="addWaterMark">添加水印</div>
  <div @click="downloadImages">下载图片</div>
  <canvas ref="canvasRef" class="canvas_container"></canvas>
</template>

<style scoped>
.canvas_container {
  /* width: 500px; */
  height: 500px;
  border: 1px solid #ccc;
}
#fileInput {
  display: none;
}
.imgList {
  display: flex;
  width: 80vw;
  height: 200px;
}
.imgItem {
  padding: 0 10px;
}
.imgItem img {
  width: 100px;
  height: 100px;
}
</style>
