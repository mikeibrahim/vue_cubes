<template>
  <div id="p5Canvas"></div>
</template>

<script>
export default {
  name: 'BoxVisualizer',
  props: {
    boxList: Array,
  },
  created: function () {
    const app = this;

    const script = function (p5) {
      let camera
      p5.setup = () => {
        var canvas = p5.createCanvas(500, 500, p5.WEBGL)
        p5.normalMaterial();
        canvas.parent("p5Canvas");
        console.log(app.boxList);
      }

      p5.draw = () => {
        p5.orbitControl(10, 10, 10);
        p5.scale(1, -1);
        p5.background(200);
        for (var box of app.boxList) {
          p5.push();
          p5.translate(box[0] * 50, box[1] * 50, box[2] * 50);
          // p5.rotateX(p5.frameCount * 0.01);
          // p5.rotateY(p5.frameCount * 0.01);
          p5.box(50);
          p5.pop();
        }
      }
    }

    const P5 = require('p5');
    new P5(script)
  }
}
</script>