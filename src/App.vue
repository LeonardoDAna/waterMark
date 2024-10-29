<script setup>
import dayjs from "dayjs";
import { nextTick, reactive, ref } from "vue";

const fileInput = ref();
const canvasRef = ref();
const waterMarkConfig = reactive({
  color: "#006AFF",
  content: "测试文本",
  type: 1,
  fontSize: 20,
});

const textContent = ref(dayjs().format("YYYY-MM-DD HH:mm:ss"));

const types = [
  { value: 1, label: "左上" },
  { value: 2, label: "右上" },
  { value: 3, label: "垂直" },
  { value: 4, label: "水平" },
];

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
      console.log(images);

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
  };
};
const addWaterMark = () => {
  let img = setupData.photoList[0];
  let proportion = img.width / img.height;
  let scale = 0.4;

  let config = {
    fontSize: waterMarkConfig.fontSize,
    text: waterMarkConfig.content,
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
  ctx.fillStyle = waterMarkConfig.color; // 黑色文字，透明度为0.5
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
  const base64Img = canvas.toDataURL("image/png");
  var a = document.createElement("a"); // 生成一个a元素
  var event = new MouseEvent("click"); // 创建一个单击事件
  a.download = dayjs().valueOf("YYYY-MM-DD HH:mm:ss"); // 设置图片名称
  a.href = base64Img; // 将生成的URL设置为a.href属性
  a.dispatchEvent(event);
};
</script>

<template>
  <div class="uploadFilesBtn" @click="upload">批量上传图片</div>
  <input
    ref="fileInput"
    type="file"
    id="fileInput"
    name="file"
    multiple
    @change="getFiles"
  />
  <div>
    <h4>图片列表</h4>
    <div class="imgList">
      <template v-for="(item, index) in setupData.photoList" :key="index">
        <div class="imgItem">
          <img :src="item.currentSrc" alt="" />
          <div class="imgItem-info">
            <div>{{ item.name }}</div>
            <div>{{ item.size }}</div>
          </div>
        </div>
      </template>
    </div>
  </div>
  <h4>操作栏</h4>
  <div class="operation_container">
    <div class="left_container">
      <el-form :model="form" label-width="auto">
        <el-form-item label="内容">
          <el-input v-model="waterMarkConfig.content" />
        </el-form-item>
        <el-form-item label="颜色">
          <el-color-picker v-model="waterMarkConfig.color" />
        </el-form-item>
        <el-form-item label="位置">
          <el-select v-model="waterMarkConfig.type" placeholder="please select your zone">
            <template v-for="item in types" :key="item">
              <el-option :label="item.label" :value="item.value" />
            </template>
          </el-select>
        </el-form-item>
        <el-form-item label="字号">
          <el-input v-model="waterMarkConfig.fontSize" />
        </el-form-item>
      </el-form>
    </div>
    <div class="right_container">
      <div class="operation_btn addBtn" @click="addWaterMark">添加水印</div>
      <div class="operation_btn downloadBtn" @click="downloadImages">下载图片</div>
      <div class="operation_btn resetBtn">重置</div>
    </div>
  </div>
  <canvas ref="canvasRef" class="canvas_container"></canvas>
</template>

<style scoped>
.operation_container {
  display: flex;
}
.left_container {
  width: 100%;
}
.right_container {
  /* width: 50%; */
  padding: 0 20px;
  display: flex;
  flex-direction: column;
  /* align-items: center; */
  justify-content: space-between;
}
.addBtn {
  background-color: #005ba5ff;
  /* margin-bottom: 20px; */
}
.downloadBtn {
  /* margin-bottom: 20px; */
  background-color: #00a54aff;
}
.operation_btn {
  width: 200px;
  height: 50px;
  line-height: 50px;
  border-radius: 8px;
  cursor: pointer;
  user-select: none;
  color: #ffff;
}
.uploadFilesBtn {
  border: 2px dashed #0077ff;
  height: 50px;
  line-height: 50px;
  cursor: pointer;
}
.resetBtn {
  background-color: #a50000ff;
}
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
  /* width: 80vw; */
  height: 100px;
  padding: 10px 0;
  background-color: #dddddd;
}
.imgItem {
  padding: 0 10px;
}
.imgItem img {
  /* width: 100px; */
  height: 100px;
}
</style>
