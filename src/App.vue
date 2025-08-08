<template>
  <div class="app">
    <canvas ref="liveCanvas"></canvas>
  </div>
</template>

<script>
import * as PIXI from 'pixi.js';
import { Live2DModel } from 'pixi-live2d-display/cubism4';

window.PIXI = PIXI;
let app;
let model;
export default {
  async mounted() {
    app = new PIXI.Application({
      view: this.$refs.liveCanvas,
      autoStart: true,
      resizeTo: window,
      backgroundAlpha: 0,
    });

    model = await Live2DModel.from('./lian/face1.model3.json');
    app.stage.addChild(model);
    model.scale.set(0.2);
    model.internalModel.coreModel.setParameterValueById("ParamMouthOpenY", 0);

    // 建立WebSocket连接
    this.ws = new WebSocket("ws://localhost:8765/");
    this.ws.onopen = () => console.log("连接成功");
    this.ws.onerror = (e) => console.log("连接失败", e);
    this.ws.onmessage = (event) => {
      try {
        const data = JSON.parse(event.data);
        if (data.volume !== undefined) {
          // 限制范围0~1
          let vol = Math.min(Math.max(data.volume, 0), 1);
          model.internalModel.coreModel.setParameterValueById("ParamMouthOpenY", vol);
        }
      } catch (e) {
        console.error("WebSocket数据解析错误", e);
      }
    };
  },
  beforeUnmount() {
    model?.destroy();
    app?.destroy();
    if (this.ws) this.ws.close();
  },
};
</script>



<style scoped>
.app{
  background-color: #ffffff;
}
header {
  line-height: 1.5;
}
</style>


