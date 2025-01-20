<template>
    <el-container>
        <el-header></el-header>
        <el-aside>
            <div id="toolBox" class="tool-box-display tool-box">
                <button class="image-button" :class="{selected:buttonType === 1}" @click="changeButtonType(1)">
                    <img src="@/assets/icon/rect.svg" alt="rect">
                </button>
                <button class="image-button" :class="{selected:buttonType === 2}" @click="changeButtonType(2)">
                    <img src="@/assets/icon/circle.svg" alt="circle">
                </button>
                <button class="image-button" :class="{selected:buttonType === 3}" @click="changeButtonType(3)">
                    <img src="@/assets/icon/ellipse.svg" alt="ellipse">
                </button>
                <button class="image-button" :class="{selected:buttonType === 4}" @click="changeButtonType(4)">
                    <img src="@/assets/icon/line.svg" alt="line">
                </button>
                <!--
                <button class="image-button" :class="{selected:buttonType === 5}" @click="changeButtonType(5)">
                    <img src="../assets/icons/polygon.svg" alt="polygon">
                </button>
                <button class="image-button" :class="{selected:buttonType === 6}" @click="changeButtonType(6)">
                    <img src="../assets/icons/polyline.svg" alt="polyline">
                </button>
                <button class="image-button" :class="{selected:buttonType === 7}" @click="changeButtonType(7)">
                    <img src="../assets/icons/path.svg" alt="path">
                </button>
                -->
            </div>
        </el-aside>

        <!-- 工具箱 -->
        <!-- svg区域 -->
        <el-main>
            <svg id="svgId" :width="whiteboardSize.width" :height="whiteboardSize.height" xmlns="http://www.w3.org/2000/svg" style="border: 1px solid #5C5C5C;position: absolute;left: 10%;top:10%">

                <!-- 矩形组 -->
                <g id="rectGroup">
                    <!-- 正在绘制的矩形 -->
                    <rect v-if="isOpt"
                          :x="currentOptRect.x" :y="currentOptRect.y" :width="currentOptRect.width" :height="currentOptRect.height"
                          :rx="currentOptRect.rx" :ry="currentOptRect.ry"
                          :fill="currentOptRect.fill"
                          :stroke="currentOptRect.stroke" :stroke-width="currentOptRect.strokeWidth"
                    >
                    </rect>

                    <!-- 已有完成的矩形 -->
                    <rect v-for="rect in rectGroupList"
                          :key="rect.id" :x="rect.x" :y="rect.y" :width="rect.width" :height="rect.height"
                          :rx="rect.rx" :ry="rect.ry"
                          :fill="rect.fill"
                          :stroke="rect.stroke" :stroke-width="rect.strokeWidth"
                    >
                    </rect>
                </g>

                <!-- 圆形组 -->
                <g id="circleGroup">
                    <!-- 正在绘制的圆形 -->
                    <circle :cx="currentOptCircle.cx" :cy="currentOptCircle.cy" :r="currentOptCircle.r"
                            :fill="currentOptCircle.fill"
                            :stroke="currentOptCircle.stroke" :stroke-width="currentOptCircle.strokeWidth"
                    >
                    </circle>

                    <!-- 已有完成的圆形 -->
                    <circle v-for="circle in circleGroupList"
                            :key="circle.id" :cx="circle.cx" :cy="circle.cy" :r="circle.r"
                            :fill="circle.fill"
                            :stroke="circle.stroke" :stroke-width="circle.strokeWidth"
                    >
                    </circle>
                </g>

                <!-- 椭圆组 -->
                <g id="ellipseGroup">
                    <!-- 正在绘制的椭圆 -->
                    <ellipse :cx="currentOptEllipse.cx" :cy="currentOptEllipse.cy" :rx="currentOptEllipse.rx" :ry="currentOptEllipse.ry"
                             :fill="currentOptEllipse.fill"
                             :stroke="currentOptEllipse.stroke" :stroke-width="currentOptEllipse.strokeWidth"
                    >
                    </ellipse>

                    <!-- 已有完成的椭圆 -->
                    <ellipse v-for="ellipse in ellipseGroupList"
                             :key="ellipse.id" :cx="ellipse.cx" :cy="ellipse.cy" :rx="ellipse.rx" :ry="ellipse.ry"
                             :fill="ellipse.fill"
                             :stroke="ellipse.stroke" :stroke-width="ellipse.strokeWidth"
                    >
                    </ellipse>
                </g>

                <!-- 直线组 -->
                <g>
                    <!-- 正在绘制的直线 -->
                    <line :x1="currentOptLine.x1" :y1="currentOptLine.y1" :x2="currentOptLine.x2" :y2="currentOptLine.y2"
                          :stroke="currentOptLine.stroke" :stroke-width="currentOptLine.strokeWidth"
                    >
                    </line>

                    <!-- 已有完成的直线 -->
                    <line v-for="line in lineGroupList"
                          :x1="line.x1" :y1="line.y1" :x2="line.x2" :y2="line.y2"
                          :stroke="line.stroke" :stroke-width="line.strokeWidth"
                    >
                    </line>
                </g>

            </svg>
        </el-main>

        <el-footer>
            <p style="font-size: 14px;font-weight: 900">坐标：({{ coordinatesTip.x }},{{ coordinatesTip.y }})</p>
        </el-footer>
    </el-container>
</template>

<script setup lang="js" name="BoardView">
import {onMounted, onUnmounted, reactive, ref} from "vue";

const coordinatesTip = ref({x: 0, y: 0})                        // 当前鼠标坐标点位提示
const coordinates = ref({})                                     // 当前鼠标坐标点位
const buttonType = ref(null)                                    // 当前按钮触发类型(矩形、圆形、椭圆......)
const whiteboardSize = reactive({width: '0px', height: '0px'})
let mouseDownType = null                                              // 鼠标按键类型(左、中、右)
const isOpt = ref(false)

let currentOptRect = ref({})                                    // 正在操作的矩形
let rectGroupList = ref([])                                     // 矩形组列表

let currentOptCircle = ref({})                                  // 正在操作的圆形
let circleGroupList = ref([])                                   // 圆形组列表

let currentOptEllipse = ref({})                                 // 正在操作的椭圆
let ellipseGroupList = ref([])                                  // 椭圆组列表

let currentOptLine = ref({})                                    // 正在操作的直线
let lineGroupList = ref([])                                     // 直线组列表

// 组件挂载时绑定事件
onMounted(() => {
    calScreenSize()
    const svgElement = document.getElementById('svgId')

    window.addEventListener('resize', calScreenSize)

    /** 禁止右键菜单弹出 */
    svgElement.addEventListener('contextmenu', (e) => {
        e.preventDefault()
    })

    /** 监听鼠标按下事件 */
    svgElement.addEventListener('mousedown', (e) => {
        e.preventDefault()
        // 判断鼠标按键
        if (e.button === 0) {
            coordinates.value = {x: e.offsetX, y: e.offsetY}
            isOpt.value = true
            mouseDownType = 0
        } // 左键
        else if (e.button === 1) mouseDownType = 1 // 中键
        else if (e.button === 2) mouseDownType = 2 // 右键
    })

    /** 监听鼠标松开事件 */
    svgElement.addEventListener('mouseup', () => {
        mouseDownType = null
        isOpt.value = false
        switch (buttonType.value) {
            case 1:
                if (currentOptRect.value.width > 0 && currentOptRect.value.height > 0) {
                    rectGroupList.value.push(currentOptRect.value)
                    currentOptRect.value = {}
                }
                break;
            case 2:
                if (currentOptCircle.value.r > 0) {
                    circleGroupList.value.push(currentOptCircle.value)
                    currentOptCircle.value = {}
                }
                break;
            case 3:
                if (currentOptEllipse.value.rx > 0 && currentOptEllipse.value.ry > 0) {
                    ellipseGroupList.value.push(currentOptEllipse.value)
                    currentOptEllipse.value = {}
                }
                break;
            case 4:
                if (currentOptLine.value.x2 - currentOptLine.value.x1 !== 0 && currentOptLine.value.y2 - currentOptLine.value.y1 !== 0) {
                    lineGroupList.value.push(currentOptLine.value)
                    currentOptLine.value = {}
                }
                break;
        }

    })

    /** 监听鼠标移动事件 */
    svgElement.addEventListener('mousemove', (e) => {
        // 更新坐标信息
        coordinatesTip.value = {x: e.offsetX, y: e.offsetY}
        if (mouseDownType === 0) mouseMoveLeft(e)
        else if (mouseDownType === 2) mouseMoveRight(e)
    })

})

/** 页面卸载操作 */
onUnmounted(() => {
    window.removeEventListener('resize', calScreenSize)
})

/** 计算屏幕宽度 */
const calScreenSize = () => {
    let browserInfo = {width: window.innerWidth, avgWidth: window.innerWidth / 24, height: window.innerHeight, avgHeight: window.innerHeight / 24}
    let avgWidth = browserInfo.avgWidth, avgHeight = browserInfo.avgHeight
    whiteboardSize.width = avgWidth * 20 + 'px'
    whiteboardSize.height = avgHeight * 20 + 'px'
}

/** 选择按钮类型操作 */
const changeButtonType = (type) => {
    if (buttonType.value === type) buttonType.value = null
    else buttonType.value = type
}

/** 鼠标左键移动事件 */
const mouseMoveLeft = (e) => {
    switch (buttonType.value) {
        case 1:
            drawRect(e)
            break;
        case 2:
            drawCircle(e)
            break;
        case 3:
            drawEllipse(e)
            break;
        case 4:
            drawLine(e)
            break;
        case 5:
            break;
    }
}

/** 矩形描画操作 */
const drawRect = (e) => {
    let startX = coordinates.value.x, startY = coordinates.value.y
    let width = e.offsetX - startX, height = e.offsetY - startY
    if (width < 0) {
        currentOptRect.value.x = e.offsetX
        currentOptRect.value.width = startX - e.offsetX
    } else {
        currentOptRect.value.x = startX
        currentOptRect.value.width = width
    }

    if (height < 0) {
        currentOptRect.value.y = e.offsetY
        currentOptRect.value.height = startY - e.offsetY
    } else {
        currentOptRect.value.y = startY
        currentOptRect.value.height = height
    }
    currentOptRect.value = {
        ...currentOptRect.value,
        id: Date.now(),
        rx: 10, ry: 10, fill: 'none', stroke: 'lightBlue', strokeWidth: 2
    }
}

/** 圆形描画操作 */
const drawCircle = (e) => {
    let startX = coordinates.value.x, startY = coordinates.value.y
    let r = e.offsetX - startX
    if (r < 0) {
        currentOptCircle.value.r = -r
    } else {
        currentOptCircle.value.r = r
    }

    currentOptCircle.value = {
        ...currentOptCircle.value,
        id: Date.now(),
        cx: startX, cy: startY, fill: 'none', stroke: 'lightBlue', strokeWidth: 2
    }
}

/** 椭圆描画操作 */
const drawEllipse = (e) => {
    let startX = coordinates.value.x, startY = coordinates.value.y
    let rx = e.offsetX - startX, ry = e.offsetY - startY
    if (rx < 0) currentOptEllipse.value.rx = -rx
    else currentOptEllipse.value.rx = rx
    if (ry < 0) currentOptEllipse.value.ry = -ry
    else currentOptEllipse.value.ry = ry
    currentOptEllipse.value = {
        ...currentOptEllipse.value,
        id: Date.now(),
        cx: startX, cy: startY, fill: 'none', stroke: 'lightBlue', strokeWidth: 2
    }
}

/** 直线描画操作 */
const drawLine = (e) => {
    let startX = coordinates.value.x, startY = coordinates.value.y
    let endX = e.offsetX, endY = e.offsetY
    currentOptLine.value = {
        id: Date.now(),
        x1: startX, y1: startY, x2: endX, y2: endY,
        fill: 'none', stroke: 'lightBlue', strokeWidth: 2
    }
}

/** 鼠标右键移动事件 */
const mouseMoveRight = (e) => {

}
</script>

<style scoped>
.tool-box-display {
    display: flex;
    flex-direction: column; /* 垂直排列 */
    align-items: center; /* 水平居中 */
    justify-content: space-between; /* 垂直方向平均分布 */
    height: 100%; /* 让容器高度填满父容器 */
    padding: 10px 0; /* 可选：给顶部和底部增加一点内边距 */
    box-sizing: border-box; /* 让 padding 包含在高度内 */
}

.tool-box {
    position: absolute;
    top: 10%;
    left: 5%;
    width: 50px;
    height: 500px;
    border-radius: 10px;
    border: 2px solid #5C5C5C;
}

.image-button {
    width: 40px;
    height: 40px;
    padding: 0;
    border: none;
    background: none;
    cursor: pointer;
    overflow: hidden;
    transition: all 0.2s ease;
}

.image-button:focus {
    outline: none; /* 去除默认的焦点边框 */
}

.image-button img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    transition: transform 0.2s ease;
}

/* 鼠标移到按钮时，按钮变小并内嵌 */
.image-button:hover img {
    transform: scale(0.9); /* 图片缩小，内嵌效果 */
}

/* 鼠标点击时，保持内嵌效果 */
.image-button:active img {
    transform: scale(0.75); /* 更小的内嵌效果 */
}

/* 按钮被选中时的内嵌效果 */
.image-button.selected img {
    transform: scale(0.75); /* 选中时缩小，保持内嵌效果 */
}

:deep(.el-header) {
    height: 20px;
}

</style>
