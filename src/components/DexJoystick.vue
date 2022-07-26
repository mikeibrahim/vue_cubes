<template>
  <div id="p5Joystick"></div>
</template>

<script>
export default {
  name: 'DexJoystick',
  props: {
    canvasSize: { type: Object, default: () => ({ x: 400, y: 400 }) }, // Canvas size in pixels
    pointerSize: { type: Number, default: () => 20 },
    objectScale: { type: Number, default: () => 1 },
    // Colors
    enabledColor: { type: String, default: () => '#3B7EDA' },
    disabledColor: { type: String, default: () => '#333333' },
    pointerColor: { type: String, default: () => '#B71C1C' },
    backgroundColor: { type: String, default: () => '#eeeeee' },
  },
  data() {
    return {
      joystick: null,
      enabled: false,
      joystickData: {
        x: 0, // [0, 1]
        y: 0, // [0, 1]
        pressed: false,
      },
    }
  },
  mounted() {
    this.renderJoystick(this)
  },
  methods: {
    renderJoystick(app) {
      // // eslint-disable-next-line @typescript-eslint/no-var-requires
      const P5 = require('p5')
      new P5((p5) => {
        // Start
        p5.setup = () => {
          app.setup(p5)
          app.createConnectButton(p5, app)
        }

        // Update every frame
        p5.draw = () => {
          p5.background(app.backgroundColor)
          app.renderJoystickMoveArea(p5)
          app.renderJoystickMovement(p5)
        }
      })
    },
    setup(p5) {
      const canvas = p5.createCanvas(this.canvasSize.x, this.canvasSize.y);
      canvas.parent("p5Joystick")
    },
    async createConnectButton(p5, app) {
      let device = null;
      await navigator.hid.getDevices().then(devices => {
        device = devices[0];
      })

      const inputReport = (event, app) => {
        const { data, device, reportId } = event
        const xRaw = data.getUint8(1)
        const yRaw = data.getUint8(3)
        const buttonRaw = data.getUint8(4)
        app.joystickData = {
          x: xRaw <= 8 ? app.scale(xRaw, 0, 8, -1, 0) : app.scale(xRaw, 8, 15, 0, 1),
          y: yRaw <= 8 ? app.scale(yRaw, 0, 8, -1, 0) : app.scale(yRaw, 8, 15, 0, 1),
          pressed: buttonRaw == 1,
        }
      }

      const setUpDevice = (device, app) => {
        app.joystick = device;
        app.enabled = true;
        if (!app.joystick.opened) {
          app.joystick.open();
        }
        app.joystick.addEventListener("inputreport", (event) => inputReport(event, app));
      }

      // Initial pairing of device
      if (!device) {
        let button = p5.createButton('Connect Joystick');
        button.parent("p5Joystick");
        button.addClass("connectButton");
        button.mousePressed(async () => {
          button.remove();
          const device = await navigator.hid.requestDevice({ filters: [{ vendorId: 0x068e }] })
          setUpDevice(device[0], app)
        });
      } else {
        setUpDevice(device, app)
      }
    },
    renderJoystickMoveArea(p5) {
      p5.push()
      p5.smooth()
      p5.noStroke()
      p5.fill(this.enabled ? this.enabledColor : this.disabledColor)
      p5.circle(p5.width / 2, p5.height / 2, p5.width)
      p5.pop()
    },
    renderJoystickMovement(p5) {
      if (!this.joystick) {
        return
      }
      let pos = p5.createVector(this.joystickData.x, this.joystickData.y)
      if (pos.mag() > 0.9) {
        pos.normalize()
        pos.mult(0.9)
      }
      const mag = pos.mag()
      pos.x = pos.x * p5.width / 2 + p5.width / 2
      pos.y = pos.y * p5.height / 2 + p5.height / 2

      p5.push()
      p5.stroke(255)
      // Center point
      p5.ellipse(p5.width / 2, p5.height / 2, this.pointerSize, this.pointerSize);
      // Connecting line
      p5.strokeWeight(Math.min(10, mag * 20))
      p5.line(p5.width / 2, p5.height / 2, pos.x, pos.y);
      p5.pop()

      // Arrow point
      p5.push()
      p5.noStroke()
      var angle = p5.atan2(p5.height / 2 - pos.y, p5.width / 2 - pos.x);
      p5.translate(pos.x, pos.y);
      p5.rotate(angle - p5.HALF_PI);
      p5.scale(Math.min(2, mag * 3) * this.objectScale)
      p5.triangle(-this.pointerSize * this.objectScale * 0.5, this.pointerSize * this.objectScale, this.pointerSize * this.objectScale * 0.5, this.pointerSize * this.objectScale, 0, -this.pointerSize * this.objectScale / 2); //draws the arrow point as a triangle
      p5.pop();
    },
    scale(number, inMin, inMax, outMin, outMax) {
      return (number - inMin) * (outMax - outMin) / (inMax - inMin) + outMin;
    }
  }
}
</script>

<style>
#p5Joystick {
  position: relative;
  border: 2px solid rgb(115, 115, 115);
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