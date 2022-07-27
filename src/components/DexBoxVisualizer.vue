<template>
  <div ref="canvas" class="p5BoxVisualizer"></div>
</template>

<script>
import P5 from 'p5'

export default {
  name: 'DexBoxVisualizer',
  props: {
    boxList: { type: Array, default: () => [] }, // Placed boxes
    objectScale: { type: Number, default: () => 75 }, // Master scale of rendered objects
    animationSpeed: { type: Number, default: () => 3 }, // Animation speed for instantiated boxes
    canvasSize: { type: Object, default: () => ({ x: 750, y: 500 }) }, // Canvas size in pixels
    groundSize: { type: Object, default: () => ({ x: 4, y: 0.05, z: 2.27 }) },
    selectedBoxError: { type: Boolean, default: () => false }, // Error state of selected box
    // Colors
    boxColor: { type: String, default: () => '#ffffff' },
    selectedColor: { type: String, default: () => '#3B7EDA' },
    errorColor: { type: String, default: () => '#B71C1C' },
    backgroundColor: { type: String, default: () => '#eeeeee' },
    groundColor: { type: String, default: () => '#333333' },
  },
  data() {
    return {
      p5: null,
      selectedBox: null,
      boxSlider: null, // Slider to control the number of rendered Boxes
      numRenderedBoxes: 0,
      animationProgress: 1,
    }
  },
  mounted() {
    this.generateBoxes()
  },
  unmounted() {
    this.p5 = null
  },
  watch: {
    boxList(newBoxList) {
      if (!newBoxList) {
        return
      }
      if (this.selectedBox) {
        if (newBoxList.length >= 2 && this.selectedBox.id === newBoxList[newBoxList.length - 2].id) {
          this.animationProgress = 0
        }
      }
      if (this.p5) {
        this.createBoxSlider(this.numRenderedBoxes === newBoxList.length - 1 ? newBoxList.length : this.numRenderedBoxes)
      }
      this.selectedBox = newBoxList[newBoxList.length - 1]
    },
  },
  methods: {
    // Uses p5 to render the desired number of boxes to the canvas
    generateBoxes() {
      this.p5 = new P5(function (p5) {
        p5.setup = this.setup // Start the sketch
        p5.draw = this.draw // Update every frame
      }.bind(this))
    },
    // Sets up canvas and camera
    setup() {
      const canvas = this.p5.createCanvas(this.canvasSize.x, this.canvasSize.y, this.p5.WEBGL);
      canvas.parent(this.$refs.canvas)
      this.p5.camera(-250, -150, 0)
      this.p5.angleMode(this.p5.DEGREES)
      this.createBoxSlider(this.boxList ? this.boxList.length : 1)
    },
    // Slider to control the number of rendered boxes
    createBoxSlider(value) {
      if (this.boxSlider) {
        this.boxSlider.remove()
      }
      this.boxSlider = this.p5.createSlider(0, this.boxList ? this.boxList.length : 1, value)
      this.boxSlider.parent(this.$refs.canvas)
      this.boxSlider.style('width', this.canvasSize.x / 4 + 'px');
      this.boxSlider.addClass("slider");
    },
    draw() {
      this.orbitControl()
      this.p5.scale(1, -1)
      this.p5.background(this.backgroundColor)
      this.renderGround()
      if (!this.boxList) {
        return
      }
      this.updateRenderedBoxes()
      this.updateAnimation()
      this.renderBoxes()
    },
    // Only allow orbiting the scene if the slider is not being interacted with
    orbitControl() {
      this.p5.orbitControl(5, 0, 0)
    },
    renderGround() {
      this.p5.push()
      this.p5.fill(this.groundColor)
      this.p5.translate((-this.groundSize.x / 2 + 1) * this.objectScale, 0, 0)
      this.p5.scale(this.groundSize.x, this.groundSize.y, this.groundSize.z)
      this.p5.box(this.objectScale)
      this.p5.pop()
    },
    // Update the scene's rendered boxes according to the boxSlider's value
    updateRenderedBoxes() {
      this.numRenderedBoxes = this.boxSlider.value()
    },
    updateAnimation() {
      if (this.animationProgress < 1) {
        // Using deltaTime to make animations framerate independent
        this.animationProgress += Math.min(this.animationSpeed * this.p5.deltaTime / 1000, 1)
      }
    },
    // Displays desired boxes, and shows a "ghost" of boxes that will be placed
    renderBoxes() {
      for (let [i, box] of this.boxList.entries()) {
        this.p5.push()
        const isSelectedBox = i === this.boxList.length - 1
        const isRenderedBox = i < this.numRenderedBoxes

        let fillColor = this.p5.color(this.boxColor)
        // Selected boxes are a different color & have instantiation animations
        if (isSelectedBox) {
          const amt = this.smooth(this.animationProgress)
          let boxColorOpacity = this.p5.color(this.selectedBoxError ? this.errorColor : this.selectedColor)
          boxColorOpacity.setAlpha(amt * 255)
          fillColor = boxColorOpacity
          this.p5.strokeWeight(amt)
        }
        // If a box is not rendered, it is a ghost box and should have lowered opacity
        if (!isRenderedBox) {
          fillColor.setAlpha(Math.min(this.p5.alpha(fillColor), 50))
          this.p5.strokeWeight(0)
        }

        // Box transformations
        this.p5.rotateX(box.rotation.x)
        this.p5.rotateZ(box.rotation.y)
        this.p5.rotateY(box.rotation.z)
        this.p5.translate(
          box.translation.x * this.objectScale * -1,
          box.translation.z * this.objectScale,
          (box.translation.y - this.groundSize.z / 2) * this.objectScale
        )
        this.p5.scale(box.size.x, box.size.z, box.size.y)
        this.p5.fill(fillColor)
        this.p5.box(this.objectScale)
        this.p5.pop()
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
.p5BoxVisualizer {
  position: relative;
  border: 2px solid rgb(115, 115, 115);
  position: relative;
}

.slider {
  position: absolute;
  top: 110%;
  left: 0;
  right: 0;
  margin-left: auto;
  margin-right: auto;
  transform: scale(2);
  z-index: 100;
}
</style>