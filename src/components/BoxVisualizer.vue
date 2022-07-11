<template>
  <div id="p5Canvas"></div>
</template>

<script>
export default {
  name: 'BoxVisualizer',
  props: {
    boxList: { type: Array, default: () => [] },
    boxColor: { type: String, default: () => '#ffffff' },
    lastBoxColor: { type: String, default: () => '#3B7EDA' },
    errorColor: { type: String, default: () => '#B71C1C' },
    lastBoxError: { type: Boolean, default: () => false },
    canvasSize: { type: Object, default: () => ({ x: 500, y: 500 }) },
    backgroundColor: { type: String, default: () => '#eeeeee' },
    groundColor: { type: String, default: () => '#333333' },
    objectScale: { type: Number, default: () => 1.4 },
  },
  mounted() {
    this.generateBoxes(this)
  },
  methods: {
    generateBoxes(app) {
      const P5 = require('p5')
      new P5((p5) => {
        // Start
        p5.setup = () => {
          var canvas = p5.createCanvas(app.canvasSize.x, app.canvasSize.y, p5.WEBGL);
          canvas.parent("p5Canvas")
          p5.camera(-250, -250, 300)
          p5.angleMode(p5.DEGREES)
        }

        // Update every frame
        p5.draw = () => {
          p5.orbitControl(5, 5, 5)
          p5.scale(1, -1)
          p5.background(app.backgroundColor)

          // Render ground
          p5.push()
          p5.fill(app.groundColor)
          p5.rectMode(p5.CENTER)
          p5.rotateX(90)
          p5.translate(0, 0, 1)
          p5.rect(0, 0, app.canvasSize.x, app.canvasSize.y)
          p5.pop()

          // Render boxes
          for (let [i, box] of app.boxList.entries()) {
            p5.push()
            const isLastBox = i === app.boxList.length - 1
            if (isLastBox) {
              p5.fill(app.lastBoxError ? app.errorColor : app.lastBoxColor)
            } else {
              p5.fill(app.boxColor)
            }
            p5.rotateX(box.rotation.x)
            p5.rotateY(box.rotation.y)
            p5.rotateZ(box.rotation.z)
            p5.translate(
              (box.translation.x + box.size.x / 2) * 50 * app.objectScale,
              (box.translation.y + box.size.y / 2) * 50 * app.objectScale,
              (box.translation.z + box.size.z / 2) * 50 * app.objectScale
            )
            p5.scale(box.size.x * app.objectScale, box.size.y * app.objectScale, box.size.z * app.objectScale)
            p5.box(50)
            p5.pop()
          }
        }
      })
    },
  }
}
</script>

<style>
#p5Canvas {
  /* border */
  border: 2px solid rgb(115, 115, 115);
}
</style>