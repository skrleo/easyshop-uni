<template>
    <canvas
        class="fire-canvas"
        :style="{width: width + 'px', height: height + 'px', 'z-index': zIndex}"
        :canvas-id="canvasId"
        @error="canvasIdErrorCallback"
    ></canvas>
</template>

<script>
/*
    浏览器的最高刷新赔率（最后一个表示1秒刷新60次）
*/
const minBrowserRefreshTime = window && (window.requestAnimationFrame || window.webkitRequestAnimationFrame || window.mozRequestAnimationFrame || window.oRequestAnimationFrame || window.msRequestAnimationFrame) || function (t) {
    setTimeout(t, 1e3 / 60)
}
const systenInfo = uni.getSystemInfoSync()
let fireCanvasBox = null
export default {
    props: {
        /*
            礼花数量（最好小于500,太多会卡顿）
        */
        particleCount: {
            type: [Number, String],
            default: 150
        },
        /* 取值	0-360
            喷发角度示意图（简单说就是喷射方向）
                    礼花（90）
            礼花（180）		礼花0
                    礼花（270）
        */
        angle: {
            type: [Number, String],
            default: 90
        },
        /*
            爆炸范围
        */
        spread: {
            type: [Number, String],
            default: 100
        },
        /*
            喷发的初始速度
        */
        startVelocity: {
            type: [Number, String],
            default: 45
        },
        /*
            喷发的衰退时间，超出canvas会被清除，跟startVelocity配合使用
        */
        decay: {
            type: [Number, String],
            default: 0.9
        },
        /*
            刷新几次消失（其实是透明度为0），这个跟间隔的刷新频率有关
        */
        ticks: {
            type: [Number, String],
            default: 150
        },
        /*
            礼花层级
        */
        zIndex: {
            type: [Number, String],
            default: 1
        },
        /*
            所有要随机的礼花颜色预选值
        */
        colors: {
            type: Array,
            default: () => ['#5BC0EB', '#2176AE', '#FDE74C', '#9BC53D', '#E55934', '#FA7921', '#FF4242']
        },
        canvasId: {
            type: String,
            default: 'fireCanvas'
        },
        /*
            canvas宽度(单位px)
        */
        width: {
            type: [Number, String],
            default: () => {
                return systenInfo.windowWidth
            }
        },
        /*
            canvas高度(单位px)
        */
        height: {
            type: [Number, String],
            default: () => {
                return systenInfo.windowHeight
            }
        },
        /*
            中心点-x
        */
        x: {
            type: [Number, String],
            default: () => {
                return systenInfo.windowWidth / 2
            }
        },
        /*
            中心点-y
        */
        y: {
            type: [Number, String],
            default: () => {
                return systenInfo.windowHeight * 0.4
            }
        }
    },
    data() {
        return {
            /*
                手机分辨率
            */
            pixelRatio: systenInfo.pixelRatio
        }
    },
    mounted() {
        this.initCanvas()
    },
    methods: {
        /*
            转16进制（颜色用）
        */
        parseInt16(t) {
            return parseInt(t, 16)
        },
        canvasIdErrorCallback(e) {
            console.error(e.detail.errMsg)
        },
        initCanvas() {
            fireCanvasBox = null
            fireCanvasBox = uni.createCanvasContext(this.canvasId, this)
            fireCanvasBox.fillRect(0, 0, this.width * this.pixelRatio, this.height * this.pixelRatio)
            fireCanvasBox.scale(this.pixelRatio, this.pixelRatio)
            fireCanvasBox.save()
            this.fireworksDraw()
        },
        fireworksDraw() {
            let ribbon = []; let // 彩带容器
                particleCount = this.particleCount; let n = null; let r = null; let a = null; let i = null
            for (; particleCount--;) {
                n = {
                    x: this.x,
                    y: this.y,
                    angle: this.angle,
                    spread: this.spread,
                    startVelocity: this.startVelocity,
                    color: this.colors[particleCount % this.colors.length],
                    ticks: this.ticks,
                    decay: this.decay
                },
                    i = 0,
                    r = n.angle * (Math.PI / 180),
                    a = n.spread * (Math.PI / 180)
                ribbon.push({// 菜单位置初始化
                    x: n.x,
                    y: n.y,
                    depth: 0.5 * Math.random() + 0.6,
                    wobble: 10 * Math.random(),
                    velocity: 0.5 * n.startVelocity + Math.random() * n.startVelocity,
                    angle2D: -r + (0.5 * a - Math.random() * a),
                    tiltAngle: Math.random() * Math.PI,
                    color: (i = (n.color + '').replace(/[^0-9a-f]/gi, ''), i.length < 6 && (i = i[0] + i[0] + i[1] + i[1] + i[2] + i[2]), {// 生成随机颜色
                        r: this.parseInt16(i.substring(0, 2)),
                        g: this.parseInt16(i.substring(2, 4)),
                        b: this.parseInt16(i.substring(4, 6))
                    }),
                    tick: 0,
                    totalTicks: n.ticks,
                    decay: n.decay,
                    random: Math.random() + 5,
                    tiltSin: 0,
                    tiltCos: 0,
                    wobbleX: 0,
                    wobbleY: 0
                })
            }
            minBrowserRefreshTime(function drawRibbon() {
                if (!fireCanvasBox) return
                fireCanvasBox.draw(),
                    fireCanvasBox.restore(),
                    ribbon = ribbon.filter((e) => {
                        e.x += Math.cos(e.angle2D) * e.velocity,
                            e.y += Math.sin(e.angle2D) * e.velocity + 5 * e.depth,
                            e.wobble += 0.1,
                            e.velocity *= e.decay,
                            e.tiltAngle += 0.02 * Math.random() + 0.12,
                            e.tiltSin = Math.sin(e.tiltAngle),
                            e.tiltCos = Math.cos(e.tiltAngle),
                            e.random = Math.random() + 4,
                            e.wobbleX = e.x + 10 * Math.cos(e.wobble) * e.depth,
                            e.wobbleY = e.y + 10 * Math.sin(e.wobble) * e.depth
                        // 开始画图
                        fireCanvasBox.fillStyle = 'rgba(' + e.color.r + ', ' + e.color.g + ', ' + e.color.b + ', ' + (1 - (e.tick++) / e.totalTicks) + ')',
                            fireCanvasBox.beginPath(),
                            fireCanvasBox.moveTo(Math.floor(e.x), Math.floor(e.y)),
                            fireCanvasBox.lineTo(Math.floor(e.wobbleX), Math.floor(e.y + e.random * e.tiltSin)),
                            fireCanvasBox.lineTo(Math.floor(e.wobbleX + e.random * e.tiltCos), Math.floor(e.wobbleY + e.random * e.tiltSin)),
                            fireCanvasBox.lineTo(Math.floor(e.x + e.random * e.tiltCos), Math.floor(e.wobbleY)),
                            fireCanvasBox.closePath(),
                            fireCanvasBox.fill()
                        return e.tick < e.totalTicks
                    })
                ribbon.length ? minBrowserRefreshTime(drawRibbon) : fireCanvasBox = null// 轮询调用或者释放掉
            }
            )
        }
    }
}
</script>

<style>
.fire-canvas {
    position: fixed;
    top: 0px;
    left: 0px;
    pointer-events: none;
}
</style>
