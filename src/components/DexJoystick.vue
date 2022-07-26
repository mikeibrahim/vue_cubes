<template>
  <div id="p5Joystick"></div>
</template>

<script>
import P5 from 'p5'

export default {
  name: 'DexJoystick',
  props: {
    canvasSize: { type: Object, default: () => ({ x: 400, y: 400 }) }, // Canvas size in pixels
    pointerSize: { type: Number, default: () => 20 },
    objectScale: { type: Number, default: () => 1 },
    connectMessage: { type: String, default: () => 'Connect Joystick' },
    // Colors
    enabledColor: { type: String, default: () => '#3B7EDA' },
    disabledColor: { type: String, default: () => '#546e7a' },
    pointerColor: { type: String, default: () => '#ffffff' },
    backgroundColor: { type: String, default: () => '#ffffff' },
  },
  data() {
    return {
      p5: null,
      connectButton: null,
      joystick: null,
      enabled: false,
      joystickData: {
        x: 0, // [-1, 1]
        y: 0, // [-1, 1]
        pressed: false,
      },
      prevPressed: false,
    }
  },
  mounted() {
    this.renderJoystick(this)
  },
  methods: {
    renderJoystick(app) {
      new P5((p5) => {
        // Start
        p5.setup = () => {
          app.setup(p5)
          app.createConnectButton()
        }

        // Update every frame
        p5.draw = () => {
          p5.background(app.backgroundColor)
          app.renderJoystickMoveArea()
          app.renderJoystickMovement()
        }
      })
    },
    setup(p5) {
      this.p5 = p5
      const canvas = this.p5.createCanvas(this.canvasSize.x, this.canvasSize.y)
      canvas.parent("p5Joystick")
    },
    createConnectButton() {
      navigator.hid.getDevices().then(devices => {
        this.connectButton?.remove()
        if (devices.length == 0) {
          this.connectButton = this.p5.createButton(this.connectMessage)
          this.connectButton.parent("p5Joystick")
          this.connectButton.addClass("connectButton")
          this.connectButton.mousePressed(() => {
            navigator.hid.requestDevice({ filters: [{ vendorId: 0x068e }] }).then(devices => {
              if (devices.length == 0) {
                console.log("No device found")
                return;
              }
              this.setUpDevice(devices[0])
              this.connectButton.remove()
            })
          });
        } else {
          this.setUpDevice(devices[0])
        }
      })
    },
    setUpDevice(device) {
      if (!this.joystick) {
        navigator.hid.addEventListener('disconnect', (event) => { this.disconnectDevice() })
      }
      this.joystick = device
      if (!this.joystick.opened) {
        this.joystick.open()
      }
      this.joystick.addEventListener("inputreport", (event) => this.inputReport())
    },
    inputReport() {
      const { data, device, reportId } = event
      const xRaw = data.getUint8(1)
      const yRaw = data.getUint8(3)
      const buttonRaw = data.getUint8(4)
      this.joystickData = {
        x: xRaw <= 8 ? this.scale(xRaw, 0, 8, -1, 0) : this.scale(xRaw, 8, 15, 0, 1),
        y: yRaw <= 8 ? this.scale(yRaw, 0, 8, -1, 0) : this.scale(yRaw, 8, 15, 0, 1),
        pressed: buttonRaw == 1,
      }
      if (this.prevPressed && !this.joystickData.pressed) {
        this.enabled = !this.enabled
      }
      this.prevPressed = this.joystickData.pressed
    },
    disconnectDevice() {
      console.log("Disconnecting from device: " + this.joystick);
      if (this.joystick.opened) {
        this.joystick.close();
      }
      this.enabled = false;
      this.createConnectButton()
    },
    renderJoystickMoveArea() {
      this.p5.push()
      this.p5.smooth()
      this.p5.noStroke()
      this.p5.fill(this.enabled ? this.enabledColor : this.disabledColor)
      this.p5.circle(this.p5.width / 2, this.p5.height / 2, this.p5.width)
      this.p5.pop()
    },
    renderJoystickMovement() {
      if (!(this.joystick && this.enabled)) {
        return
      }
      let pos = this.p5.createVector(this.joystickData.x, this.joystickData.y)
      if (pos.mag() > 0.9) {
        pos.normalize()
        pos.mult(0.9)
      }
      const mag = pos.mag()
      pos.x = pos.x * this.p5.width / 2 + this.p5.width / 2
      pos.y = pos.y * this.p5.height / 2 + this.p5.height / 2

      this.p5.push()
      this.p5.noStroke()
      this.p5.fill(this.pointerColor)

      // Center point
      this.p5.ellipse(this.p5.width / 2, this.p5.height / 2, this.pointerSize * this.objectScale, this.pointerSize * this.objectScale)

      // Connecting line
      this.p5.push()
      this.p5.stroke(this.pointerColor)
      this.p5.strokeWeight(Math.min(7, mag * 10) * this.objectScale * this.objectScale)
      this.p5.line(this.p5.width / 2, this.p5.height / 2, pos.x, pos.y)
      this.p5.pop()

      // Arrow point
      this.p5.push()
      const angle = this.p5.atan2(this.p5.height / 2 - pos.y, this.p5.width / 2 - pos.x)
      this.p5.translate(pos.x, pos.y)
      this.p5.rotate(angle - this.p5.HALF_PI)
      this.p5.scale(Math.min(2, mag * 3) * this.objectScale)
      this.p5.triangle(
        -this.pointerSize * this.objectScale * 0.5,
        this.pointerSize * this.objectScale,
        this.pointerSize * this.objectScale * 0.5,
        this.pointerSize * this.objectScale,
        0,
        -this.pointerSize * this.objectScale / 2
      )
      this.p5.pop()
      this.p5.pop()
    },
    scale(number, inMin, inMax, outMin, outMax) {
      return (number - inMin) * (outMax - outMin) / (inMax - inMin) + outMin
    }
  }
}
</script>

<style>
#p5Joystick {
  position: relative;
  /* border: 2px solid rgb(115, 115, 115); */
  position: relative;
}

.connectButton {
  position: absolute;
  top: 50%;
  left: 0;
  right: 0;
  width: 30%;
  height: 10%;
  margin-left: auto;
  margin-right: auto;
  transform: scale(2);
  z-index: 100;
}
</style>