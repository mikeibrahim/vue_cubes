<template>
  <div id="p5Canvas"></div>
</template>

<script>
export default {
  name: 'BoxConveyorVisualizer',
  props: {
    boxList: { type: Array, default: () => [] }, // Queue of boxes
    boxCount: { type: Number, default: () => 0 }, // Number of boxes to show on conveyor
    animationSpeed: { type: Number, default: () => 1 }, // Animation speed for instantiated boxes
    objectScale: { type: Number, default: () => 75 }, // Master scale of rendered objects
    canvasSize: { type: Object, default: () => ({ x: 750, y: 750 }) }, // Canvas size in pixels
    conveyorSize: { type: Object, default: () => ({ x: 4, y: 0.25, z: 1 }) },
    groundSize: { type: Object, default: () => ({ x: 10, y: 0.05, z: 10 }) },
    selectedBoxError: { type: Boolean, default: () => false }, // Error state of selected box
    // Colors
    boxColor: { type: String, default: () => '#ffffff' },
    selectedBoxColor: { type: String, default: () => '#3B7EDA' },
    selectedBoxErrorColor: { type: String, default: () => '#B71C1C' },
    conveyorColor: { type: String, default: () => '#9ecbff' },
    groundColor: { type: String, default: () => '#333333' },
    backgroundColor: { type: String, default: () => '#eeeeee' },
  },
  data() {
    return {
      renderedBoxes: [],
      selectedBox: null,
      animationProgress: 0,
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
          p5.orbitControl(5, 5, 0)
          p5.scale(1, -1)
          p5.background(app.backgroundColor)
          app.renderGround(p5)
          app.renderConveyor(p5)
          app.updateRenderedBoxes()
          app.updateAnimation(p5)
          app.renderBoxes(p5)
        }
      }
      )
    },
    // Sets up canvas and camera
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
      p5.scale(this.groundSize.x, this.groundSize.y, this.groundSize.z)
      p5.box(this.objectScale)
      p5.pop()
    },
    renderConveyor(p5) {
      p5.push()
      p5.fill(this.conveyorColor)
      p5.strokeWeight(0.5)
      p5.translate(0, this.conveyorSize.y * this.objectScale / 2, 0)
      p5.scale(this.conveyorSize.x, this.conveyorSize.y, this.conveyorSize.z)
      p5.box(this.objectScale)
      p5.pop()
    },
    // Handles animation logic & display of boxes
    updateRenderedBoxes() {
      // Initialize the renderedBoxes on first render
      if (!this.selectedBox && this.boxList.length > 0) {
        this.selectedBox = this.boxList[0]
        this.renderedBoxes = this.boxList.slice(0, this.boxCount)
      }

      // If the first box is getting removed from the queue
      if (this.boxList[0] != this.selectedBox) {
        this.renderedBoxes = [this.selectedBox].concat(this.boxList.slice(0, this.boxCount))
        this.selectedBox = this.boxList[0]
        this.animationProgress = 0
      }

      // After animation progress is complete, 
      if (this.animationProgress >= 1) {
        this.animationProgress = 0
        this.renderedBoxes.shift()
      }
    },
    updateAnimation(p5) {
      if (this.animationProgress < 1 && this.renderedBoxes[0] != this.selectedBox) {
        this.animationProgress = this.animationProgress + this.animationSpeed * p5.deltaTime / 1000
      }
    },
    // Displays boxes that are in the queue
    renderBoxes(p5) {
      for (let [i, box] of this.renderedBoxes.entries()) {
        p5.push()
        let fillColor = p5.color(this.boxColor)
        // Selected boxes are rendered in a different color
        if (box == this.selectedBox) {
          fillColor = this.selectedBoxError ? this.selectedBoxErrorColor : this.selectedBoxColor
        }
        // Boxes in the front/back of queue have opacity animations
        else if (i == 0 || i == this.boxCount) {
          let amt = this.smooth(this.animationProgress)
          amt = i == 0 ? 1 - amt : amt
          let boxColorOpacity = p5.color(this.boxColor)
          boxColorOpacity.setAlpha(amt * 255)
          fillColor = boxColorOpacity
          p5.strokeWeight(amt)
        }

        // Rendering the box
        const order = this.boxCount - i - 1 + this.smooth(this.animationProgress)
        const bounds = this.conveyorSize.x * this.objectScale * 0.8
        const xPos = bounds / (this.boxCount - 1) * order - bounds / 2
        const yPos = this.conveyorSize.y * this.objectScale + box.size.y * this.objectScale / 2
        p5.translate(xPos, yPos, 0)
        p5.scale(box.size.x, box.size.y, box.size.z)
        p5.fill(fillColor)
        p5.box(this.objectScale)
        p5.pop()
      }
    },
    // Smooth interpolation of range [0, 1] for animation purposes
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