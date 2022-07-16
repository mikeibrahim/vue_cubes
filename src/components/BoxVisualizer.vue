<template>
  <div id="p5Canvas"></div>
</template>

<script>
export default {
  name: 'BoxVisualizer',
  props: {
    boxList: { type: Array, default: () => [] },
    boxColor: { type: String, default: () => '#ffffff' },
    selectedColor: { type: String, default: () => '#3B7EDA' },
    errorColor: { type: String, default: () => '#B71C1C' },
    backgroundColor: { type: String, default: () => '#eeeeee' },
    groundColor: { type: String, default: () => '#333333' },
    selectedBoxError: { type: Boolean, default: () => false },
  },
  data() {
    return {
      objectScale: 75,
      selectedBox: null,
      boxSpeed: 3,
      boxProgress: 0,
      canvasSize: { x: 750, y: 750 },
    }
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
          app.setup(p5)
        }

        // Update every frame
        p5.draw = () => {
          p5.orbitControl(5, 5, 5)
          p5.scale(1, -1)
          p5.background(app.backgroundColor)
          app.renderGround(p5)
          app.updateSelectedBox()
          app.updateBoxProgress(p5)
          app.renderBoxes(p5)
        }
      })
    },
    setup(p5) {
      const canvas = p5.createCanvas(this.canvasSize.x, this.canvasSize.y, p5.WEBGL);
      canvas.parent("p5Canvas")
      p5.camera(-250, -250, 300)
      p5.angleMode(p5.DEGREES)
    },
    renderGround(p5) {
      p5.push()
      p5.fill(this.groundColor)
      p5.rectMode(p5.CENTER)
      p5.rotateX(90)
      p5.translate(0, 0, 1)
      p5.rect(0, 0, this.canvasSize.x, this.canvasSize.y)
      p5.pop()
    },
    updateSelectedBox() {
      const lastBox = this.boxList[this.boxList.length - 1]
      if (this.selectedBox != lastBox) {
        this.selectedBox = lastBox
        this.boxProgress = 0
      }
    },
    updateBoxProgress(p5) {
      if (this.boxProgress < 1) {
        this.boxProgress += this.boxSpeed * p5.deltaTime / 1000
      }
    },
    renderBoxes(p5) {
      for (let [i, box] of this.boxList.entries()) {
        p5.push()
        const isSelectedBox = i === this.boxList.length - 1
        if (isSelectedBox) {
          const amt = this.smooth(this.boxProgress)
          let boxColorOpacity = p5.color(this.selectedBoxError ? this.errorColor : this.selectedColor)
          boxColorOpacity.setAlpha(amt * 255)
          p5.fill(boxColorOpacity)
          p5.strokeWeight(amt)
        } else {
          p5.fill(this.boxColor)
        }
        p5.rotateX(box.rotation.x)
        p5.rotateZ(box.rotation.y)
        p5.rotateY(box.rotation.z)
        p5.translate(
          box.translation.x * this.objectScale * -1,
          box.translation.z * this.objectScale,
          box.translation.y * this.objectScale
        )
        p5.scale(box.size.x, box.size.z, box.size.y)
        p5.box(this.objectScale)
        p5.pop()
      }
    },
    smooth(x) {
      return 0.5 * Math.cos(Math.PI * x + Math.PI) + 0.5
    }
  }
}
</script>

<style>
#p5Canvas {
  /* border */
  border: 2px solid rgb(115, 115, 115);
}
</style>