<template>
  <div id="p5Canvas"></div>
</template>

<script>
export default {
  name: 'BoxConveyorVisualizer',
  props: {
    boxList: { type: Array, default: () => [] },
    boxCount: { type: Number, default: () => 0 },
    boxColor: { type: String, default: () => '#ffffff' },
    selectedBoxColor: { type: String, default: () => '#3B7EDA' },
    selectedBoxErrorColor: { type: String, default: () => '#B71C1C' },
    conveyorColor: { type: String, default: () => '#9ecbff' },
    groundColor: { type: String, default: () => '#333333' },
    backgroundColor: { type: String, default: () => '#eeeeee' },
    selectedBoxError: { type: Boolean, default: () => false },
  },
  data() {
    return {
      renderedBoxes: [],
      selectedBox: null,
      boxSpeed: 1,
      boxProgress: 0,
      objectScale: 75,
      conveyorSize: { x: 4, y: 0.25, z: 1 },
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
          app.renderConveyor(p5)
          app.updateRenderedBoxes()
          app.updateBoxProgress(p5)
          app.renderBoxes(p5)
        }
      }
      )
    },
    setup(p5) {
      const canvas = p5.createCanvas(this.canvasSize.x, this.canvasSize.y, p5.WEBGL);
      canvas.parent("p5Canvas")
      p5.camera(-250, -250, 300)
      p5.angleMode(p5.DEGREES)
      this.previousBoxList = this.boxList.slice()
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
    renderConveyor(p5) {
      p5.push()
      p5.fill(this.conveyorColor)
      p5.translate(0, this.conveyorSize.y * this.objectScale / 2, 0)
      p5.scale(this.conveyorSize.x, this.conveyorSize.y, this.conveyorSize.z)
      p5.box(this.objectScale)
      p5.pop()
    },
    updateRenderedBoxes() {
      // initialize the renderedBoxes
      if (!this.selectedBox && this.boxList.length > 0) {
        this.selectedBox = this.boxList[0]
        this.renderedBoxes = this.boxList.slice(0, this.boxCount)
      }

      // if the first box is coming out of the queue
      if (this.boxList[0] != this.selectedBox) {
        this.renderedBoxes = [this.selectedBox].concat(this.boxList.slice(0, this.boxCount))
        this.selectedBox = this.boxList[0]
        this.boxProgress = 0
      }

      if (this.boxProgress >= 1) {
        this.boxProgress = 0
        this.renderedBoxes.shift()
      }
    },
    updateBoxProgress(p5) {
      if (this.boxProgress < 1 && this.renderedBoxes[0] != this.selectedBox) {
        this.boxProgress = this.boxProgress + this.boxSpeed * p5.deltaTime / 1000
      }
    },
    renderBoxes(p5) {
      for (let [i, box] of this.renderedBoxes.entries()) {
        p5.push()
        if (box == this.selectedBox) {
          p5.fill(this.selectedBoxError ? this.selectedBoxErrorColor : this.selectedBoxColor)
        } else if (i == 0 || i == this.boxCount) {
          let amt = this.smooth(this.boxProgress)
          amt = i == 0 ? 1 - amt : amt
          let boxColorOpacity = p5.color(this.boxColor)
          boxColorOpacity.setAlpha(amt * 255)
          p5.fill(boxColorOpacity)
          p5.strokeWeight(amt)
        } else {
          p5.fill(this.boxColor)
        }
        const order = this.boxCount - i - 1 + this.smooth(this.boxProgress)
        const bounds = this.conveyorSize.x * this.objectScale * 0.8
        const xPos = bounds / (this.boxCount - 1) * order - bounds / 2
        const yPos = this.conveyorSize.y * this.objectScale + box.size.y * this.objectScale / 2
        p5.translate(xPos, yPos, 0)
        p5.scale(box.size.x, box.size.y, box.size.z)
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