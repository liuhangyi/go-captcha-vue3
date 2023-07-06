<template>
    <div class="wg-cap-wrap">
        <div class="wg-cap-wrap__header">
            <span v-if="props.imageBase64">请在下图<em>依次</em>点击：</span>
            <span v-else>验证码加载中，请稍等！</span>
            <img v-if="props.thumbBase64" :src="props.thumbBase64" alt=" " class="wg-cap-wrap__thumb">
        </div>
        <div ref="ImgRef" class="wg-cap-wrap__body">
            <div v-if="showDetectAnimation" class="loading-wrap"></div>
            <img v-if="props.imageBase64" :src="props.imageBase64" alt=" " class="wg-cap-wrap__picture"
                 @click="handleClickPos($event)">
            <img v-else
                 :style="loadingStyle"
                 alt="正在加载中..."
                 class="wg-cap-wrap__loading"
                 src="data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz4KPHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hsaW5rIiBzdHlsZT0ibWFyZ2luOiBhdXRvOyBiYWNrZ3JvdW5kOiByZ2JhKDI0MSwgMjQyLCAyNDMsIDApOyBkaXNwbGF5OiBibG9jazsgc2hhcGUtcmVuZGVyaW5nOiBhdXRvOyIgd2lkdGg9IjY0cHgiIGhlaWdodD0iNjRweCIgdmlld0JveD0iMCAwIDEwMCAxMDAiIHByZXNlcnZlQXNwZWN0UmF0aW89InhNaWRZTWlkIj4KICA8Y2lyY2xlIGN4PSI1MCIgY3k9IjM2LjgxMDEiIHI9IjEzIiBmaWxsPSIjM2U3Y2ZmIj4KICAgIDxhbmltYXRlIGF0dHJpYnV0ZU5hbWU9ImN5IiBkdXI9IjFzIiByZXBlYXRDb3VudD0iaW5kZWZpbml0ZSIgY2FsY01vZGU9InNwbGluZSIga2V5U3BsaW5lcz0iMC40NSAwIDAuOSAwLjU1OzAgMC40NSAwLjU1IDAuOSIga2V5VGltZXM9IjA7MC41OzEiIHZhbHVlcz0iMjM7Nzc7MjMiPjwvYW5pbWF0ZT4KICA8L2NpcmNsZT4KPC9zdmc+">
            <template v-for="(dot, key) in dots" :key="key">
                <div :style="`top: ${(dot.y / relativeProportion) - 11}px; left:${(dot.x / relativeProportion) - 11}px;`"
                     class="wg-cap-wrap__dot">
                    <span>{{ dot.index }}</span>
                </div>
            </template>
            <div v-if="showAlertMsg" :style="{backgroundColor:alertMsgBgColor}" class="alert-msg-wrap">{{ alertMsg }}
            </div>
        </div>
        <div class="wg-cap-wrap__footer">
            <div class="wg-cap-wrap__ico">
                <img alt="关闭"
                     src="data:image/svg+xml;base64,PHN2ZyB0PSIxNjI2NjE0NDM5NDIzIiBjbGFzcz0iaWNvbiIgdmlld0JveD0iMCAwIDEwMjQgMTAyNCIgdmVyc2lvbj0iMS4xIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHAtaWQ9Ijg2NzUiIHdpZHRoPSIyMDAiIGhlaWdodD0iMjAwIj48cGF0aCBkPSJNNTEyIDIzLjI3MjcyN2MyNjkuOTE3MDkxIDAgNDg4LjcyNzI3MyAyMTguODEwMTgyIDQ4OC43MjcyNzMgNDg4LjcyNzI3M2E0ODYuNjMyNzI3IDQ4Ni42MzI3MjcgMCAwIDEtODQuOTQ1NDU1IDI3NS40MDk0NTUgNDYuNTQ1NDU1IDQ2LjU0NTQ1NSAwIDAgMS03Ni44NDY1NDUtNTIuNTI2NTQ2QTM5My41NDE4MTggMzkzLjU0MTgxOCAwIDAgMCA5MDcuNjM2MzY0IDUxMmMwLTIxOC41MDc2MzYtMTc3LjEyODcyNy0zOTUuNjM2MzY0LTM5NS42MzYzNjQtMzk1LjYzNjM2NFMxMTYuMzYzNjM2IDI5My40OTIzNjQgMTE2LjM2MzYzNiA1MTJzMTc3LjEyODcyNyAzOTUuNjM2MzY0IDM5NS42MzYzNjQgMzk1LjYzNjM2NGEzOTUuMTcwOTA5IDM5NS4xNzA5MDkgMCAwIDAgMTI1LjQ0LTIwLjI5MzgxOSA0Ni41NDU0NTUgNDYuNTQ1NDU1IDAgMCAxIDI5LjQ4NjU0NSA4OC4yOTY3MjhBNDg4LjI2MTgxOCA0ODguMjYxODE4IDAgMCAxIDUxMiAxMDAwLjcyNzI3M0MyNDIuMDgyOTA5IDEwMDAuNzI3MjczIDIzLjI3MjcyNyA3ODEuOTE3MDkxIDIzLjI3MjcyNyA1MTJTMjQyLjA4MjkwOSAyMy4yNzI3MjcgNTEyIDIzLjI3MjcyN3ogbS0xMTUuMiAzMDcuNzEyTDUxMiA0NDYuMTM4MTgybDExNS4yLTExNS4yYTQ2LjU0NTQ1NSA0Ni41NDU0NTUgMCAxIDEgNjUuODE1MjczIDY1Ljg2MTgxOEw1NzcuODYxODE4IDUxMmwxMTUuMiAxMTUuMmE0Ni41NDU0NTUgNDYuNTQ1NDU1IDAgMSAxLTY1Ljg2MTgxOCA2NS44MTUyNzNMNTEyIDU3Ny44NjE4MThsLTExNS4yIDExNS4yYTQ2LjU0NTQ1NSA0Ni41NDU0NTUgMCAxIDEtNjUuODE1MjczLTY1Ljg2MTgxOEw0NDYuMTM4MTgyIDUxMmwtMTE1LjItMTE1LjJhNDYuNTQ1NDU1IDQ2LjU0NTQ1NSAwIDEgMSA2NS44NjE4MTgtNjUuODE1MjczeiIgcC1pZD0iODY3NiIgZmlsbD0iIzcwNzA3MCI+PC9wYXRoPjwvc3ZnPg=="
                     @click="handleCloseEvent">
                <img alt="刷新"
                     src="data:image/svg+xml;base64,PHN2ZyB0PSIxNjI2NjE0NDk5NjM4IiBjbGFzcz0iaWNvbiIgdmlld0JveD0iMCAwIDEwMjQgMTAyNCIgdmVyc2lvbj0iMS4xIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHAtaWQ9IjEzNjAiIHdpZHRoPSIyMDAiIGhlaWdodD0iMjAwIj48cGF0aCBkPSJNMTg3LjQ1NiA0MjUuMDI0YTMzNiAzMzYgMCAwIDAgMzY4LjM4NCA0MjAuMjI0IDQ4IDQ4IDAgMCAxIDEyLjU0NCA5NS4xNjggNDMyIDQzMiAwIDAgMS00NzMuNjY0LTU0MC4xNmwtNTcuMjgtMTUuMzZhMTIuOCAxMi44IDAgMCAxLTYuMjcyLTIwLjkyOGwxNTkuMTY4LTE3OS40NTZhMTIuOCAxMi44IDAgMCAxIDIyLjE0NCA1Ljg4OGw0OC4wNjQgMjM1LjA3MmExMi44IDEyLjggMCAwIDEtMTUuODA4IDE0LjkxMmwtNTcuMjgtMTUuMzZ6TTgzNi40OCA1OTkuMDRhMzM2IDMzNiAwIDAgMC0zNjguMzg0LTQyMC4yMjQgNDggNDggMCAxIDEtMTIuNTQ0LTk1LjE2OCA0MzIgNDMyIDAgMCAxIDQ3My42NjQgNTQwLjE2bDU3LjI4IDE1LjM2YTEyLjggMTIuOCAwIDAgMSA2LjI3MiAyMC45MjhsLTE1OS4xNjggMTc5LjQ1NmExMi44IDEyLjggMCAwIDEtMjIuMTQ0LTUuODg4bC00OC4wNjQtMjM1LjA3MmExMi44IDEyLjggMCAwIDEgMTUuODA4LTE0LjkxMmw1Ny4yOCAxNS4zNnoiIGZpbGw9IiM3MDcwNzAiIHAtaWQ9IjEzNjEiPjwvcGF0aD48L3N2Zz4="
                     @click="handleRefreshEvent">
            </div>
            <div class="wg-cap-wrap__btn">
                <button @click="handleConfirmEvent">确认</button>
            </div>
        </div>
    </div>
</template>

<script setup>
import {computed, nextTick, onBeforeUnmount, onMounted, ref, watch} from "vue";

const props = defineProps(
    {
        originalWidth: {
            type: Number,
            default: 300
        },
        originalHeight: {
            type: Number,
            default: 240
        },
        calcPosType: {
            type: String,
            default: 'dom',
            validator: value => ['dom', 'screen'].includes(value)
        },
        maxDot: {
            type: Number,
            default: 5
            // validator: value => value > 10
        },
        imageBase64: String,
        thumbBase64: String,
        successMsgBarColor: {
            type: String,
            default: 'rgba(21,122,25,0.79)',
        },
        failMsgBarColor: {
            type: String,
            default: 'rgba(255,0,0,0.65)',
        },
    }
)

const emit = defineEmits(['close', 'refresh', 'confirm'])

const dots = ref([])
const imageBase64Code = ref('')
const thumbBase64Code = ref('')

const ImgRef = ref(null)
const RealImageWidth = ref(0)

const originalWidth = ref(props.originalWidth)
const originalHeight = ref(props.originalHeight)


const showAlertMsg = ref(false)
const showDetectAnimation = ref(false)
const alertMsg = ref("")
const alertMsgBgColor = ref("")

watch(() => {
    return props.thumbBase64
}, (val) => {
    dots.value = []
    thumbBase64Code.value = val
})

watch(ImgRef, (val) => {
    calcRealImageWidth()
})

function calcRealImageWidth() {
    if (ImgRef.value) {
        const {imgWidth} = getDomXY(ImgRef.value)
        RealImageWidth.value = imgWidth
    }
}

const relativeProportion = computed(() => {
    return (originalWidth.value / RealImageWidth.value)
})

function getImageSize(url) {
    return new Promise(function (resolve, reject) {
        let image = new Image();
        image.onload = function () {
            resolve({
                width: image.width,
                height: image.height
            });
        };
        image.onerror = function () {
            reject(new Error('failed to load image'));
        };
        image.src = url;
    });
}

watch(() => {
    return props.imageBase64
}, async (val) => {
    dots.value = []
    try {
        let size = await getImageSize(val)
        if (size) {
            originalWidth.value = size.width
            originalHeight.value = size.height
        }
    }catch (e) {

    }
    imageBase64Code.value = val
})

const loadingStyle = computed(() => {
    return {width: `40%`, height: `${RealImageWidth.value * (originalHeight.value / originalWidth.value)}px`}
})

function handleCloseEvent() {
    emit('close')
    dots.value = []
    imageBase64Code.value = ''
    thumbBase64Code.value = ''
}

function handleRefreshEvent() {
    dots.value = []
    emit('refresh')
}

function handleCallbackMessage(message) {
    showDetectAnimation.value = false
    showAlertMsg.value = true
    alertMsg.value = message
}

function cleanMsg(ms) {
    setTimeout(
        () => {
            showAlertMsg.value = false
            alertMsg.value = ""
        }, ms
    )
}

function OkCallback(message) {
    handleCallbackMessage(message)
    alertMsgBgColor.value = props.successMsgBarColor
    cleanMsg(800)
}

function FailCallback(message) {
    handleCallbackMessage(message)
    alertMsgBgColor.value = props.failMsgBarColor
    cleanMsg(1000)
}

function handleConfirmEvent() {
    showDetectAnimation.value = true
    emit('confirm', dots.value, OkCallback, FailCallback)
}

function handleClickPos(ev) {
    if (dots.value.length >= props.maxDot) {
        return
    }
    const e = ev || window.event
    e.preventDefault()
    const dom = e.currentTarget

    const {domX, domY} = getDomXY(dom)
    // ===============================================
    // @notice 如 getDomXY 不准确可尝试使用 calcLocationLeft 或 calcLocationTop
    // const domX = this.calcLocationLeft(dom)
    // const domY = this.calcLocationTop(dom)
    // ===============================================

    let mouseX = (navigator.appName === 'Netscape') ? e.pageX : e.x + document.body.offsetTop
    let mouseY = (navigator.appName === 'Netscape') ? e.pageY : e.y + document.body.offsetTop

    if (props.calcPosType === 'screen') {
        mouseX = (navigator.appName === 'Netscape') ? e.clientX : e.x
        mouseY = (navigator.appName === 'Netscape') ? e.clientY : e.y
    }

    // 计算点击的相对位置
    const xPos = mouseX - domX
    const yPos = mouseY - domY

    // 转整形
    const xp = parseInt(xPos.toString())
    const yp = parseInt(yPos.toString())

    dots.value.push({
        x: xp * relativeProportion.value,
        y: yp * relativeProportion.value,
        index: dots.value.length + 1
    })
    return false
}

/**
 * @Description: 找到元素的屏幕位置
 * @param el
 */
function calcLocationLeft(el) {
    let tmp = el.offsetLeft
    let val = el.offsetParent
    while (val != null) {
        tmp += val.offsetLeft
        val = val.offsetParent
    }
    return tmp
}

/**
 * @Description: 找到元素的屏幕位置
 * @param el
 */
function calcLocationTop(el) {
    let tmp = el.offsetTop
    let val = el.offsetParent
    while (val != null) {
        tmp += val.offsetTop
        val = val.offsetParent
    }
    return tmp
}

/**
 * @Description: 找到元素的屏幕位置
 * @param dom
 */
function getDomXY(dom) {
    let x = 0
    let y = 0
    let w = 0
    let h = 0
    if (dom.getBoundingClientRect) {
        let box = dom.getBoundingClientRect();
        w = box.width
        h = box.height
        let D = document.documentElement;
        x = box.left + Math.max(D.scrollLeft, document.body.scrollLeft) - D.clientLeft;
        y = box.top + Math.max(D.scrollTop, document.body.scrollTop) - D.clientTop
    } else {
        while (dom !== document.body) {
            x += dom.offsetLeft
            y += dom.offsetTop
            dom = dom.offsetParent
        }
    }
    return {
        domX: x,
        domY: y,
        imgWidth: w,
        imgHeight: h,
    }
}

onMounted(() => {
    nextTick(() => {
        calcRealImageWidth()
    })
    window.addEventListener("resize", calcRealImageWidth)
})

onBeforeUnmount(() => {
    window.removeEventListener("resize", calcRealImageWidth)
})
</script>

<style>
.wg-cap-wrap {
    width: 100%;
    min-width:155px;
    height: 100%;
    background: #ffffff;

    -webkit-touch-callout: none;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
}

.wg-cap-wrap__header {
    min-height: 50px;
    width: 100%;
    font-size: 15px;
    padding-bottom: 5px;
    position: relative;

    display: -webkit-box;
    display: -webkit-flex;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-align: center;
    -webkit-align-items: center;
    -ms-flex-align: center;
    align-items: center;
    flex-wrap: wrap;
}

.wg-cap-wrap__header span {
    padding-right: 5px;
}

.wg-cap-wrap__header span em {
    padding: 0 3px;
    font-weight: bold;
    color: #3e7cff;
    font-style: normal;
}

.wg-cap-wrap__header .wg-cap-wrap__image {
    -webkit-border-radius: 5px;
    -moz-border-radius: 5px;
    border-radius: 5px;
    overflow: hidden;
    text-align: center;
    line-height: 1;
}

.wg-cap-wrap__header .wg-cap-wrap__thumb {
    min-width: 150px;
    text-align: center;
    line-height: 1;
    max-height: 100%;
    border-style: solid;
    border-color: #76839b;
    border-width: 1px;
}

.wg-cap-wrap__header .wg-cap-wrap__thumb.wg-cap-wrap__hidden {
    display: none;
}

.wg-cap-wrap__body {
    position: relative;
    display: -webkit-box;
    display: -moz-box;
    display: -ms-flexbox;
    display: -webkit-flex;
    display: flex;
    background: #34383e;
    margin: auto;
    -webkit-border-radius: 5px;
    -moz-border-radius: 5px;
    border-radius: 5px;
    overflow: hidden;
    align-items: center;
    justify-content: center;
}

.wg-cap-wrap__body .wg-cap-wrap__picture {
    position: relative;
    z-index: 10;
    width: 100%;
}

.wg-cap-wrap__body .wg-cap-wrap__picture.wg-cap-wrap__hidden {
    display: none;
}

.wg-cap-wrap__body .wg-cap-wrap__loading {
    z-index: 1;
    width: 40%;
    text-align: center;
}

.wg-cap-wrap__body .wg-cap-wrap__loading img {
    width: 50%;
}

.wg-cap-wrap__body .wg-cap-wrap__dot {
    position: absolute;
    z-index: 10;
    width: 22px;
    height: 22px;
    color: #cedffe;
    background: #3e7cff;
    border: 2px solid #f7f9fb;
    line-height: 20px;
    text-align: center;
    -webkit-border-radius: 22px;
    -moz-border-radius: 22px;
    border-radius: 22px;
    cursor: default;
}

.wg-cap-wrap__footer {
    width: 100%;
    height: 40px;
    color: #34383e;
    display: -webkit-box;
    display: -webkit-flex;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-align: center;
    -webkit-align-items: center;
    -ms-flex-align: center;
    align-items: center;
    padding-top: 15px;
}

.wg-cap-wrap__footer .wg-cap-wrap__ico {
    flex: 1;
}

.wg-cap-wrap__footer .wg-cap-wrap__ico img {
    width: 24px;
    height: 24px;
    color: #34383e;
    margin: 0 5px;
    cursor: pointer;
}

.wg-cap-wrap__footer .wg-cap-wrap__btn {
    width: 120px;
    height: 40px;
}

.wg-cap-wrap__footer .wg-cap-wrap__btn button {
    width: 100%;
    height: 40px;
    letter-spacing: 2px;
    text-align: center;
    padding: 9px 15px;
    font-size: 15px;
    -webkit-border-radius: 5px;
    -moz-border-radius: 5px;
    border-radius: 5px;
    display: inline-block;
    line-height: 1;
    white-space: nowrap;
    cursor: pointer;
    color: #fff;
    background-color: #409eff;
    border: 1px solid #409eff;
    -webkit-appearance: none;
    box-sizing: border-box;
    outline: none;
    margin: 0;
    transition: .1s;
    font-weight: 500;
    -moz-user-select: none;
    -webkit-user-select: none;
}

.wg-cap-wrap__footer .wg-cap-wrap__btn button:hover {
    background: #66b1ff;
    border-color: #66b1ff;
    color: #fff;
}

.alert-msg-wrap {
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;

    z-index: 1000;

    font-size: 15px;
    color: white;
    text-align: center;
    padding: 2px;
}

.loading-wrap {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 9999;

    font-size: 15px;
    color: white;
    text-align: center;
    padding: 2px;

    background: linear-gradient(120deg, rgba(240, 242, 245, 0.30) 25%, rgba(240, 242, 245, 0.70) 37%, rgba(240, 242, 245, 0.20) 63%);
    background-size: 400% 100%;
    animation: skeleton-loading 0.7s ease-in-out infinite;
}

@keyframes skeleton-loading {
    0% {
        background-position: 100% 50%
    }

    to {
        background-position: 0 50%
    }
}

</style>
