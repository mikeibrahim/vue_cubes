<template>
  <div id="p5BoxVisualizer"></div>
</template>

<script>
export default {
  name: 'DexBoxVisualizer',
  props: {
    boxList: { type: Array, default: () => [] }, // Placed boxes
    objectScale: { type: Number, default: () => 75 }, // Master scale of rendered objects
    animationSpeed: { type: Number, default: () => 3 }, // Animation speed for instantiated boxes
    canvasSize: { type: Object, default: () => ({ x: 750, y: 500 }) }, // Canvas size in pixels
    groundSize: { type: Object, default: () => ({ x: 5, y: 0.05, z: 2.27 }) },
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
      selectedBox: null,
      boxSlider: null, // Slider to control the number of rendered Boxes
      numRenderedBoxes: 0,
      animationProgress: 1,
    }
  },
  mounted() {
    this.generateBoxes(this)
  },
  methods: {
    // Uses p5 to render the desired number of boxes to the canvas
    generateBoxes(app) {
      // // eslint-disable-next-line @typescript-eslint/no-var-requires
      const P5 = require('p5')
      new P5((p5) => {
        // Start the sketch
        p5.setup = () => {
          app.setup(p5)
          app.createBoxSlider(p5, app.boxList ? app.boxList.length : 1)
        }

        // Update every frame
        p5.draw = () => {
          app.orbitControl(p5)
          p5.scale(1, -1)
          p5.background(app.backgroundColor)
          app.renderGround(p5)
          if (!app.boxList) {
            return
          }
          app.updateRenderedBoxes(p5)
          app.updateAnimation(p5)
          app.renderBoxes(p5)
        }
      })
    },
    // Sets up canvas and camera
    setup(p5) {
      const canvas = p5.createCanvas(this.canvasSize.x, this.canvasSize.y, p5.WEBGL);
      canvas.parent("p5BoxVisualizer")
      p5.camera(-250, -150, 0)
      p5.angleMode(p5.DEGREES)
    },
    // Slider to control the number of rendered boxes
    createBoxSlider(p5, value) {
      if (this.boxSlider) {
        this.boxSlider.remove()
      }
      this.boxSlider = p5.createSlider(0, this.boxList ? this.boxList.length : 1, value)
      this.boxSlider.parent(document.getElementById('p5BoxVisualizer'))
      this.boxSlider.style('width', p5.width / 4 + 'px');
      this.boxSlider.addClass("slider");
    },
    // Only allow orbiting the scene if the slider is not being interacted with
    orbitControl(p5) {
      p5.orbitControl(5, 0, 0)
    },
    renderGround(p5) {
      p5.push()
      p5.fill(this.groundColor)
      p5.scale(this.groundSize.x, this.groundSize.y, this.groundSize.z)
      p5.box(this.objectScale)
      p5.pop()
    },
    // Update the scene's rendered boxes according to the boxSlider's value
    updateRenderedBoxes(p5) {
      const lastBox = this.boxList[this.boxList.length - 1] // Last placed box
      // Initialize selectedBox on first render
      if (!this.selectedBox && lastBox) {
        this.selectedBox = lastBox
      }

      this.numRenderedBoxes = this.boxSlider.value()

      // When a new box is placed, update selected box & restart animation
      if (this.selectedBox && lastBox && this.selectedBox.id != lastBox.id) {
        this.selectedBox = lastBox
        this.animationProgress = 0
        // Only stay at the slider's current value if it is not maxed out, otherwise follow the last box
        this.createBoxSlider(p5, this.numRenderedBoxes == this.boxList.length - 1 ? this.boxList.length : this.numRenderedBoxes)
      }
    },
    updateAnimation(p5) {
      if (this.animationProgress < 1) {
        // Using deltaTime to make animations framerate independent
        this.animationProgress += Math.min(this.animationSpeed * p5.deltaTime / 1000, 1)
      }
    },
    // Displays desired boxes, and shows a "ghost" of boxes that will be placed
    renderBoxes(p5) {
      for (let [i, box] of this.boxList.entries()) {
        p5.push()
        const isSelectedBox = i === this.boxList.length - 1
        const isRenderedBox = i < this.numRenderedBoxes

        let fillColor = p5.color(this.boxColor)
        // Selected boxes are a different color & have instantiation animations
        if (isSelectedBox) {
          const amt = this.smooth(this.animationProgress)
          let boxColorOpacity = p5.color(this.selectedBoxError ? this.errorColor : this.selectedColor)
          boxColorOpacity.setAlpha(amt * 255)
          fillColor = boxColorOpacity
          p5.strokeWeight(amt)
        }
        // If a box is not rendered, it is a ghost box and should have lowered opacity
        if (!isRenderedBox) {
          fillColor.setAlpha(Math.min(p5.alpha(fillColor), 50))
          p5.strokeWeight(0)
        }

        // Box transformations
        p5.rotateX(box.rotation.x)
        p5.rotateZ(box.rotation.y)
        p5.rotateY(box.rotation.z)
        p5.translate(
          box.translation.x * this.objectScale * -1,
          box.translation.z * this.objectScale,
          (box.translation.y - this.groundSize.z / 2) * this.objectScale
        )
        p5.scale(box.size.x, box.size.z, box.size.y)
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
#p5BoxVisualizer {
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
}
</style>