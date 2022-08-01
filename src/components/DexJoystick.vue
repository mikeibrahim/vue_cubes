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
        connectMessage: { type: String, default: () => 'Connect' },
        dataCallback: { type: Function, default: () => () => { } },
        triggerCallback: { type: Function, default: () => () => { } },
        // Colors
        enabledColor: { type: String, default: () => '#3B7EDA' },
        disabledColor: { type: String, default: () => '#546e7a' },
        pointerColor: { type: String, default: () => '#ffffff' },
        backgroundColor: { type: String, default: () => '#ffffff' },
    },
    data() {
        return {
            p5: null, // Reference to p5 instance (avoids redundant props)
            connectButton: null,
            joystick: null,
            enabled: false,
            joystickData: {
                x: 0, // Inputs remapped to [-1, 1]
                y: 0, // Inputs remapped to [-1, 1]
                pressed: false, // APEM controller button
            },
            prevPressed: false, // Used to detect joystick toggle
        }
    },
    mounted() {
        this.initP5Canvas()
    },
    methods: {
        // Create p5 instance and set up callbacks
        initP5Canvas() {
            new P5(function (p5) {
                this.p5 = p5
                p5.setup = this.setup
                p5.draw = this.draw
            }.bind(this))
        },
        // p5.js callback, called on object startup
        setup() {
            const canvas = this.p5.createCanvas(this.canvasSize.x, this.canvasSize.y)
            canvas.parent("p5Joystick")
            this.createConnectButton()
        },
        // Establish a connection to the APEM controller
        createConnectButton() {
            // Get a list of available controllers that are already connected
            navigator.hid.getDevices().then(devices => {
                this.connectButton?.remove() // Remove old connection button if it exists
                if (devices.length == 0) {
                    this.connectButton = this.p5.createButton(this.connectMessage)
                    // font size
                    this.connectButton.style('font-size', this.pointerSize / 2 + 'px')
                    this.connectButton.parent("p5Joystick")
                    this.connectButton.addClass("connectButton")
                    this.connectButton.mousePressed(() => {
                        // Get a list of available controllers that are not already connected
                        // Vendor id queries only the APEM controller
                        navigator.hid.requestDevice({ filters: [{ vendorId: 0x068e }] }).then(devices => {
                            if (devices.length == 0) {
                                console.log("No device found")
                                return
                            }
                            this.setUpDevice(devices[0])
                            this.connectButton.remove()
                        })
                    })
                } else {
                    this.setUpDevice(devices[0])
                }
            })
        },
        setUpDevice(device) {
            // Make sure to only establish the disconnection callback once
            if (!this.joystick) {
                navigator.hid.addEventListener('disconnect', (event) => { this.disconnectDevice() })
            }
            this.joystick = device
            // Open device stream to recieve data
            if (!this.joystick.opened) {
                this.joystick.open()
            }
            this.joystick.addEventListener("inputreport", (event) => this.inputReport(event))
        },
        // Callback for when data is received from the APEM controller
        inputReport(event) {
            const { data, device, reportId } = event
            // Data structure for APEM input: [x delta, x position [0, 15], y delta, y position [0, 15], APEM button pressed {0, 1}]
            const xRaw = data.getUint8(1)
            const yRaw = data.getUint8(3)
            const buttonRaw = data.getUint8(4)

            this.joystickData = {
                // Resting state for both x and y raw is at position 8
                // Since the controller's resting state is offset by 1, there needs to be two separate remaps
                x: xRaw <= 8 ? this.scale(xRaw, 0, 8, -1, 0) : this.scale(xRaw, 8, 15, 0, 1), // Rescale values to [-1, 1] range
                y: yRaw <= 8 ? this.scale(yRaw, 0, 8, -1, 0) : this.scale(yRaw, 8, 15, 0, 1), // Rescale values to [-1, 1] range
                pressed: buttonRaw == 1,
            }

            // When the button is released, toggle the enabled state
            if (this.prevPressed && !this.joystickData.pressed) {
                this.enabled = !this.enabled
                this.triggerCallback()
            }
            this.prevPressed = this.joystickData.pressed

            this.dataCallback({
                x: this.enabled ? this.joystickData.x : 0,
                y: this.enabled ? this.joystickData.y : 0,
            })
        },
        // On device disconnection, close stream and prompt user to reconnect
        disconnectDevice() {
            console.log("Disconnecting from device: " + this.joystick)
            if (this.joystick.opened) {
                this.joystick.close()
            }
            this.enabled = false
            this.createConnectButton()
        },
        // p5.js callback, runs every frame
        draw() {
            this.p5.background(this.backgroundColor)
            this.renderJoystickMoveArea()
            this.renderJoystickMovement()
        },
        // Graphic for joystick movement bounds
        renderJoystickMoveArea() {
            this.p5.push()
            this.p5.smooth()
            this.p5.noStroke()
            this.p5.fill(this.enabled ? this.enabledColor : this.disabledColor)
            this.p5.circle(this.p5.width / 2, this.p5.height / 2, this.p5.width)
            this.p5.pop()
        },
        // Graphic for joystick input
        renderJoystickMovement() {
            // Only render joystick if activated
            if (!(this.joystick && this.enabled)) {
                return
            }

            // Normalize joystick vector to stay within circular bounds
            let pos = this.p5.createVector(this.joystickData.x, this.joystickData.y)
            if (pos.mag() > 0.9) {
                pos.normalize()
                pos.mult(0.9)
            }
            const mag = pos.mag() // Used for dynamic scaling of joystick graphic

            // Translate [-1, 1] joystick input to be centered and scaled to canvas size
            const center = this.p5.createVector(this.p5.width / 2, this.p5.height / 2)
            pos.x = pos.x * center.x + center.x
            pos.y = pos.y * center.y + center.y

            // Draw joystick graphic
            this.p5.push()
            this.p5.noStroke()
            this.p5.fill(this.pointerColor)

            // Center point
            this.p5.ellipse(center.x, center.y, this.pointerSize, this.pointerSize)

            // Connecting line
            this.p5.push()
            this.p5.stroke(this.pointerColor)
            this.p5.strokeWeight(Math.min(0.6, mag) * this.pointerSize)
            this.p5.line(center.x, center.y, pos.x, pos.y)
            this.p5.pop()

            // Arrow point
            const angle = this.p5.atan2(center.y - pos.y, center.x - pos.x)
            this.p5.translate(pos.x, pos.y)
            this.p5.rotate(angle - this.p5.HALF_PI)
            this.p5.scale(Math.min(2, mag * 3))
            this.p5.triangle(
                -this.pointerSize * 0.5,
                this.pointerSize,
                this.pointerSize * 0.5,
                this.pointerSize,
                0,
                -this.pointerSize / 2
            )
            this.p5.pop()
        },
        // Scale a value from one range to another
        scale(number, inMin, inMax, outMin, outMax) {
            return (number - inMin) * (outMax - outMin) / (inMax - inMin) + outMin
        }
    }
}
</script>

<style>
#p5Joystick {
    position: relative;
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