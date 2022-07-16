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
      currentBoxSlider: null,
      renderedBoxes: [],
      selectedBox: null,
      boxSpeed: 3,
      boxProgress: 1,
      objectScale: 75,
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
          app.createCurrentBoxSlider(p5)
        }

        // Update every frame
        p5.draw = () => {
          app.orbitControl(p5)
          p5.scale(1, -1)
          p5.background(app.backgroundColor)
          app.renderGround(p5)
          app.updateRenderedBoxes()
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
    createCurrentBoxSlider(p5) {
      this.currentBoxSlider = p5.createSlider(0, this.boxList.length, this.boxList.length)
      this.currentBoxSlider.parent(document.getElementById('p5Canvas'))
      this.currentBoxSlider.position(p5.width / 4, p5.height * 7 / 8)
      this.currentBoxSlider.style('width', p5.width / 2 + 'px');
      this.currentBoxSlider.elt.mouseIsOver = false
      this.currentBoxSlider.elt.onmouseover = function () { this.mouseIsOver = true }
      this.currentBoxSlider.elt.onmouseout = function () { this.mouseIsOver = false }
    },
    orbitControl(p5) {
      // if the mouse is not inside the box slider
      if (!this.currentBoxSlider.elt.mouseIsOver) {
        p5.orbitControl(5, 5, 5)
      }
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
    updateRenderedBoxes() {
      const lastBox = this.boxList[this.boxList.length - 1]
      if (!this.selectedBox && lastBox) {
        this.selectedBox = lastBox
      }

      if (this.selectedBox != lastBox) {
        this.selectedBox = lastBox
        this.boxProgress = 0
      }

      this.renderedBoxes = this.boxList.slice(0, this.currentBoxSlider.value())
    },
    updateBoxProgress(p5) {
      if (this.boxProgress < 1) {
        this.boxProgress += Math.min(this.boxSpeed * p5.deltaTime / 1000, 1)
      }
    },
    renderBoxes(p5) {
      for (let [i, box] of this.renderedBoxes.entries()) {
        p5.push()
        const isSelectedBox = i === this.renderedBoxes.length - 1
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
  position: relative;
  border: 2px solid rgb(115, 115, 115);
}
</style>