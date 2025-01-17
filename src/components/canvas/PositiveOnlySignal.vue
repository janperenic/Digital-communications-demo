<template>
    <h2 class="font-semibold text-xl">{{ title }}</h2>
    <canvas-container>
        <div :id="canvas_id"></div>
    </canvas-container>
</template>

<script>
import P5 from 'p5';
import CanvasContainer from "@/components/global/CanvasContainer";
import '@/types.js';

export default {
    name: "PositiveOnlySignal",
    components: {CanvasContainer},
    data() {
        return {
            p5: null,
            previousValue: 1,
            /** @type {Coordinates} */
            offset: {
                x: 0,
                y: 125,
            }
        }
    },
    mounted() {
        // Initiate new P5 instance and create canvas
        this.p5 = new P5((p5) => {
            this.$c.setup(p5);
            p5.draw = () => {
                p5.stroke(0);

                /**
                 * Code bellow draws the axis lines and labels them
                 */
                p5.background(255);
                const [canvasDimensions, canvasPadding] = [this.$c.dimensions, this.$c.canvasPadding];

                p5.strokeWeight(2);

                // Top-bottom line
                p5.line(canvasPadding, canvasPadding, canvasPadding, canvasDimensions.y - canvasPadding);

                p5.strokeWeight(1);
                // y-axis labels are passed through vertical_pool array, but they are missing zero, so we need to push it in
                const yAxisLabels = [...this.vertical_pool];
                yAxisLabels.splice(2, 0, 0);
                // Draw left-right lines on y-axis and label each fifth
                for(let i = 0; i <= 20; i++) {
                    // Make each fifth line labeled and wider
                    if(i % 5 === 0) {
                        p5.text(`${yAxisLabels[Math.trunc(i / 5)]}`.substring(0, 4), canvasPadding - 40, canvasPadding + i * 10 + 3);
                        p5.strokeWeight(2);
                        p5.line(canvasPadding - 5, canvasPadding + i * 10, canvasPadding + 5, canvasPadding + i * 10);
                        p5.strokeWeight(1);
                        continue;
                    }
                    p5.line(canvasPadding - 5, canvasPadding + i * 10, canvasPadding + 5, canvasPadding + i * 10);
                }
                // Y axis label
                p5.text(this.isModulatedText, canvasPadding - 5, canvasPadding / 2);
                // X axis label
                p5.text('t', canvasDimensions.x - 30, canvasDimensions.y / 2 + 3);
                p5.strokeWeight(2);
                this.$c.drawArrow(p5, p5.createVector(canvasPadding, canvasDimensions.y / 2), p5.createVector(canvasDimensions.x - canvasPadding, canvasDimensions.y / 2), 'black', 7, 0);

                p5.fill(1);
                p5.triangle(50, 42, 46, 50, 54, 50);

                /**
                 * Start drawing the actual chart
                 */
                p5.noFill();
                p5.stroke(this.$c.colors[this.color_id]);
                p5.strokeWeight(2);

                // Draw the shape
                p5.beginShape();
                this.normalizedData.forEach((y, x) => {
                    if (this.is_binary) {
                        // When value changes, start drawing on previous index to prevent skewed lines
                        p5.vertex((this.previousValue !== y) ? x - 1 + canvasPadding : x + canvasPadding,  y * (this.offset.y - canvasPadding / 2) + this.offset.y + canvasPadding / 2)
                        this.previousValue = y;
                    } else {
                        p5.vertex(x + canvasPadding, y * (this.offset.y - canvasPadding / 2) + this.offset.y + canvasPadding / 2);
                    }
                });
                p5.endShape();
            }
            p5.removeCanvas = () => p5.remove();
        }, this.canvas_id);
    },
    unmounted() {
        // Remove canvas, otherwise P5 object stays in memory
        this.p5.removeCanvas();
    },
    computed: {
        normalizedData() {
            const max = Math.max(...this.data.map(el => Math.abs(el)));
            const data = [...this.data];
            data.length= 600;
            return data.map(el => {
                if(this.type === 'pam4') {
                    const sign = Math.sign(el);
                    return (Math.abs(el) === 1) ? el / max - sign * 0.83 : el / max;
                }
                return el / max;
            });
        },
        isModulatedText() {
            return (this.is_modulated) ? 'y(t)' : 'x(t)';
        }
    },
    props: {
        data: {
            type: Array,
            required: true
        },
        title: {
            type: String,
            required: true
        },
        canvas_id: {
            type: String,
            required: true
        },
        is_binary: {
            type: Boolean,
            required: false,
            default: false
        },
        color_id: {
            type: Number,
            required: false,
            default: 0
        },
        is_modulated: {
            type: Boolean,
            required: false,
            default: false
        },
        vertical_pool: {
            type: Array,
            required: false,
            default: null
        },
        type: {
            type: String,
            required: false,
            default: ''
        }
    }
}
</script>