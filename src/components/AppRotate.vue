<template>
  <div class="wrap">
    <div class="painting">
      <div class="angle">
        <p>
          <strong>旋转角度：{{ rotationAngle }}°</strong>
        </p>
        <p>
          <strong>重合角度：{{ coincidentAngle }}°</strong>
        </p>
      </div>
      <div class="images">
        <div class="images-wrap">
          <div
            class="line"
            :style="`transform: rotate(${rotationAngle}deg);`"
          ></div>
          <img
            class="img-graph"
            :src="itemList[active].url"
            alt=""
            :style="`transform: rotate(${rotationAngle}deg);`"
          />
        </div>
        <div class="images-wrap">
          <img class="img-frame" :src="itemList[active].frame" alt="" />
          <div
            class="textwrap"
            :style="`transform: rotate(${rotationAngle / 2}deg);`"
          >
            <span>{{ rotationAngle }}°</span>
          </div>
          <div class="arcwrap">
            <!-- <canvas class="arc"></canvas> -->
            <!-- <svg class="arc">
              <path
                :d="`M ${arcParams.sx} ${arcParams.sy} A ${arcParams.rx} ${arcParams.ry} ${arcParams.xAxisRotation} ${arcParams.largeArcFlag} ${arcParams.sweepFlag} ${arcParams.x} ${arcParams.y}`"
                fill="none"
                stroke-width="0.2rem"
                stroke="#7C83FD"
                stroke-dasharray="8"
                stroke-dashoffset="0"
                style="
                  transform: rotate(-90deg);
                  transform-origin: center;
                  transform-box: fill-box;
                "
              />
            </svg> -->
          </div>
        </div>
      </div>
    </div>

    <div class="controller">
      <ul class="list">
        <li
          :class="active === index ? 'item active' : 'item'"
          v-for="(item, index) in itemList"
          :key="index"
          @click="handleChangeItem(item, index)"
        >
          <img :src="item.thumb" alt="" />
        </li>
      </ul>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { watch, ref, onMounted, nextTick } from "vue";
import { ElMessage } from "element-plus";
import "element-plus/es/components/message/style/css";

interface IItem {
  id: number;
  anglesNum: number;
  thumb: any;
  url: any;
  frame: any;
}

const itemList = ref<Array<IItem>>([
  {
    id: 1,
    anglesNum: 2,
    thumb: require("@/assets/images/icon2.png"),
    url: require("@/assets/images/2.png"),
    frame: require("@/assets/images/bg2.png"),
  },
  {
    id: 2,
    anglesNum: 3,
    thumb: require("@/assets/images/icon3.png"),
    url: require("@/assets/images/3.png"),
    frame: require("@/assets/images/bg3.png"),
  },
  {
    id: 3,
    anglesNum: 4,
    thumb: require("@/assets/images/icon4.png"),
    url: require("@/assets/images/4.png"),
    frame: require("@/assets/images/bg4.png"),
  },
  {
    id: 4,
    anglesNum: 5,
    thumb: require("@/assets/images/icon5.png"),
    url: require("@/assets/images/5.png"),
    frame: require("@/assets/images/bg5.png"),
  },
  {
    id: 5,
    anglesNum: 6,
    thumb: require("@/assets/images/icon6.png"),
    url: require("@/assets/images/6.png"),
    frame: require("@/assets/images/bg6.png"),
  },
]);

let active = ref<number>(0);

let rotationAngle = ref<number>(0);

let coincidentAngle = ref<number>(0);

interface IPoint {
  x: number;
  y: number;
}
let center = ref<IPoint>({
  x: 0,
  y: 0,
});

const handleChangeItem = (item: IItem, index: number) => {
  rotationAngle.value = 0;
  coincidentAngle.value = 0;
  active.value = index;
  flag.value = false;
};

const handleGetQuadrant = (center: IPoint, current: IPoint): number => {
  if (current.x > center.x && current.y > center.y) {
    return 4;
  } else if (current.x > center.x && current.y < center.y) {
    return 1;
  } else if (current.x < center.x && current.y < center.y) {
    return 2;
  } else if (current.x < center.x && current.y > center.y) {
    return 3;
  } else if (current.x == center.x && current.y == center.y) {
    return 0;
  } else {
    return -1;
  }
};

const handleGetDirection = (p1: IPoint, p2: IPoint): any => {
  if (p2.x > p1.x && p2.y < p2.y) {
    return "northeast";
  } else if (p2.x > p1.x && p2.y > p1.y) {
    return "southeast";
  } else if (p2.x < p1.x && p2.y > p1.y) {
    return "southwest";
  } else if (p2.x < p1.x && p2.y < p1.y) {
    return "northwest";
  } else if (p2.x > p1.x && p2.y == p1.y) {
    return "east";
  } else if (p2.x < p1.x && p2.y == p1.y) {
    return "west";
  } else if (p2.x == p1.x && p2.y < p1.y) {
    return "north";
  } else if (p2.x == p1.x && p2.y > p1.y) {
    return "south";
  } else if (p2.x == p1.x && p2.y == p1.y) {
    return "origin";
  } else {
    return "";
  }
};

//   var arc = Math.sin(((2 * Math.PI) / 360) * deg);
//   if (sweepFlag == 1) {
//     x = startX + r * Math.sin(((2 * Math.PI) / 360) * deg);
//     y = startY + r - r * Math.cos(((2 * Math.PI) / 360) * deg);
//   } else {
//     x = startX - r * Math.sin(((2 * Math.PI) / 360) * deg);
//     y = startY + r - r * Math.cos(((2 * Math.PI) / 360) * deg);
//   }

const lastPoint = ref<IPoint>({
  x: 0,
  y: 0,
});

const handleShowElMessage = () => {
  ElMessage({
    showClose: false,
    message: "已经找到了足够的重合角度，快看看有什么规律吧！",
    type: "success",
    duration: 2000,
    grouping: true,
  });
};

const flag = ref<boolean>(false);

watch(
  rotationAngle,
  (val, prevVal) => {
    console.log(val, prevVal);
    if (flag.value) {
      if (active.value === 0) {
        if (val % 180 === 0) {
          coincidentAngle.value = val;
          handleShowElMessage();
        }
      } else if (active.value === 1) {
        if (val % 120 === 0) {
          coincidentAngle.value = val;
          handleShowElMessage();
        }
      } else if (active.value === 2) {
        if (val % 90 === 0) {
          coincidentAngle.value = val;
          handleShowElMessage();
        }
      } else if (active.value === 3) {
        if (val % 72 === 0) {
          coincidentAngle.value = val;
          handleShowElMessage();
        }
      } else if (active.value === 4) {
        if (val % 60 === 0) {
          coincidentAngle.value = val;
          handleShowElMessage();
        }
      }
    }
    handleClearCanvas();
    handleDrawArc();
  },
  {
    immediate: false,
    deep: false,
  }
);

const handleGetAngle = (start: IPoint, end: IPoint): number => {
  const diff_x = end.x - start.x;
  const diff_y = end.y - start.y;
  return (360 * Math.atan(diff_y / diff_x)) / (2 * Math.PI);
};

let arcParams = ref<any>({
  // sx: 0,
  // sy: 0,
  // rx: 0,
  // ry: 0,
  // xAxisRotation: 0,
  // largeArcFlag: 0,
  // sweepFlag: 0,
  // x: 0,
  // y: 0,
  x: 0,
  y: 0,
  radius: 0,
  startAngle: 0,
  endAngle: 0,
  anticlockwise: false,
});

const handleClearCanvas = (): void => {
  try {
    let canvas: any = document.querySelector(".canvas");
    canvas && canvas.remove();
  } catch (err) {
    console.log(err);
  }
};

const handleDrawArc = (): void => {
  // var degree = 1; // 表示 1°
  // var radians = degree * (Math.PI / 180); // 0.0175弧度
  // await nextTick();
  const dom: any = document.querySelector(".arcwrap");
  let canvas: any = document.createElement("canvas");
  const rect: any = dom.getBoundingClientRect();
  console.log(rect);
  console.log(rect.width, rect.height);
  canvas.classList.add("canvas");
  canvas.width = rect.width;
  canvas.height = rect.height;
  canvas.style.transform = "rotate(-90deg)";
  dom.appendChild(canvas);
  // arcParams.value.sx = (1 - Math.sin(360 * Math.PI)) * (rect.width / 2);
  // arcParams.value.sy = (1 + Math.cos(360 * Math.PI)) * (rect.width / 2);
  // arcParams.value.rx = rect.width / 2;
  // arcParams.value.ry = rect.width / 2;
  // arcParams.value.xAxisRotation = 0;
  // arcParams.value.largeArcFlag = 1;
  // arcParams.value.sweepFlag = 1;
  // arcParams.value.x = (1 + Math.sin(360 * Math.PI)) * (rect.width / 2);
  // arcParams.value.y = (1 + Math.cos(360 * Math.PI)) * (rect.width / 2);
  arcParams.value.x = rect.width / 2;
  arcParams.value.y = rect.height / 2;
  arcParams.value.radius = rect.width / 2;
  arcParams.value.startAngle = 0; // (3 / 2) * Math.PI
  arcParams.value.endAngle = rotationAngle.value * (Math.PI / 180);
  arcParams.value.anticlockwise = rotationAngle.value < 0 ? true : false;
  // if (rotationAngle.value < 0) {
  //   arcParams.value.anticlockwise = true;
  // } else if (rotationAngle.value > 0) {
  //   arcParams.value.anticlockwise = false;
  // }
  const ctx: any = canvas.getContext("2d");
  ctx.beginPath();
  ctx.arc(
    arcParams.value.x,
    arcParams.value.y,
    arcParams.value.radius,
    arcParams.value.startAngle,
    arcParams.value.endAngle,
    arcParams.value.anticlockwise
  );
  // ctx.lineWidth = 8;
  ctx.setLineDash([8, 8]);
  ctx.stroke();
  console.log(arcParams, ctx);
};

const handleComputedcenter = () => {
  // 计算中心点
  let target: any = document.querySelector(".img-frame");
  center.value.x =
    target.getBoundingClientRect().x + target.getBoundingClientRect().width / 2;
  center.value.y =
    target.getBoundingClientRect().y +
    target.getBoundingClientRect().height / 2;
};

onMounted(() => {
  // 计算中心点
  handleComputedcenter();

  const graph: any = document.querySelector(".img-graph");

  let isMoving = false;

  let handleMouseMove: any = null;

  graph.addEventListener("mousedown", (event: any) => {
    console.log("mouse down");
    isMoving = true;

    // 计算中心点
    handleComputedcenter();

    graph.addEventListener(
      "mousemove",
      (handleMouseMove = (event: any) => {
        flag.value = true;

        // 获取象限
        const quadrant = handleGetQuadrant(center.value, {
          x: event.clientX,
          y: event.clientY,
        });
        console.log(quadrant);

        // 上下
        // const area =
        //   event.layerY < target.getBoundingClientRect().height / 2
        //     ? "top"
        //     : "bottom";
        // console.log(area);

        // 计算方向
        const direction = handleGetDirection(lastPoint.value, {
          x: event.clientX,
          y: event.clientY,
        });
        // console.log(direction);

        // rotationAngle.value = Math.round(
        //   // Math.atan2返回值介于-PI~PI
        //   Math.atan2(
        //     center.value.y - event.pageY,
        //     center.value.x - event.pageX
        //   ) *
        //     (180 / Math.PI) -
        //     90
        // );

        if (isMoving) {
          if (quadrant === 1) {
            if (direction.includes("east")) {
              if (rotationAngle.value > -360 && rotationAngle.value < 360) {
                rotationAngle.value += 1;
              } else if (rotationAngle.value === -360) {
                rotationAngle.value += 1;
              }
            } else if (direction.includes("west")) {
              if (rotationAngle.value > -360 && rotationAngle.value < 360) {
                rotationAngle.value -= 1;
              } else if (rotationAngle.value === 360) {
                rotationAngle.value -= 1;
              }
            } else if (direction.includes("north")) {
              if (rotationAngle.value > -360 && rotationAngle.value < 360) {
                rotationAngle.value -= 1;
              } else if (rotationAngle.value === 360) {
                rotationAngle.value -= 1;
              }
            } else if (direction.includes("south")) {
              if (rotationAngle.value > -360 && rotationAngle.value < 360) {
                rotationAngle.value += 1;
              }
            }
          } else if (quadrant === 2) {
            if (direction.includes("east")) {
              if (rotationAngle.value > -360 && rotationAngle.value < 360) {
                rotationAngle.value += 1;
              } else if (rotationAngle.value === -360) {
                rotationAngle.value += 1;
              }
            } else if (direction.includes("west")) {
              if (rotationAngle.value > -360 && rotationAngle.value < 360) {
                rotationAngle.value -= 1;
              } else if (rotationAngle.value === 360) {
                rotationAngle.value -= 1;
              }
            } else if (direction.includes("north")) {
              if (rotationAngle.value > -360 && rotationAngle.value < 360) {
                rotationAngle.value += 1;
              }
            } else if (direction.includes("south")) {
              if (rotationAngle.value > -360 && rotationAngle.value < 360) {
                rotationAngle.value -= 1;
              } else if (rotationAngle.value === 360) {
                rotationAngle.value -= 1;
              }
            }
          } else if (quadrant === 3) {
            if (direction.includes("east")) {
              if (rotationAngle.value > -360 && rotationAngle.value < 360) {
                rotationAngle.value -= 1;
              } else if (rotationAngle.value === 360) {
                rotationAngle.value -= 1;
              }
            } else if (direction.includes("west")) {
              if (rotationAngle.value > -360 && rotationAngle.value < 360) {
                rotationAngle.value += 1;
              }
            } else if (direction.includes("north")) {
              if (rotationAngle.value > -360 && rotationAngle.value < 360) {
                rotationAngle.value += 1;
              } else if (rotationAngle.value === -360) {
                rotationAngle.value += 1;
              }
            } else if (direction.includes("south")) {
              if (rotationAngle.value > -360 && rotationAngle.value < 360) {
                rotationAngle.value -= 1;
              } else if (rotationAngle.value === 360) {
                rotationAngle.value -= 1;
              }
            }
          } else if (quadrant === 4) {
            if (direction.includes("east")) {
              if (rotationAngle.value > -360 && rotationAngle.value < 360) {
                rotationAngle.value -= 1;
              } else if (rotationAngle.value === 360) {
                rotationAngle.value -= 1;
              }
            } else if (direction.includes("west")) {
              if (rotationAngle.value > -360 && rotationAngle.value < 360) {
                rotationAngle.value += 1;
              } else if (rotationAngle.value === -360) {
                rotationAngle.value += 1;
              }
            } else if (direction.includes("north")) {
              if (rotationAngle.value > -360 && rotationAngle.value < 360) {
                rotationAngle.value -= 1;
              } else if (rotationAngle.value === 360) {
                rotationAngle.value -= 1;
              }
            } else if (direction.includes("south")) {
              if (rotationAngle.value > -360 && rotationAngle.value < 360) {
                rotationAngle.value += 1;
              } else if (rotationAngle.value === -360) {
                rotationAngle.value += 1;
              }
            }
          }
        }
        lastPoint.value.x = event.clientX;
        lastPoint.value.y = event.clientY;
      })
    );
  });

  graph.addEventListener("mouseup", (event: any) => {
    console.log("mouse up");
    isMoving = false;
    graph.removeEventListener("mousemove", handleMouseMove);
  });

  graph.addEventListener("mouseleave", (event: any) => {
    console.log("mouse leave");
    isMoving = false;
    graph.removeEventListener("mousemove", handleMouseMove);
  });
});
</script>

<style scoped lang="scss">
$itemNum: 5;

.wrap {
  width: 100%;
  height: 100%;
  padding: 5% 1%;
  box-sizing: border-box;
  display: flex;
  flex-flow: row nowrap;

  .painting {
    flex-grow: 1;
    background-color: #bccee3;
    border-radius: 2rem;
    font-size: 2rem;

    .angle {
      margin-left: 1rem;
      font-size: 4rem;
    }

    .images {
      position: relative;

      .images-wrap {
        width: 56rem;
        height: 56rem;
        margin: 0 auto;
        padding: 0;
        position: absolute;
        top: 0;
        left: calc(50% - 28rem);

        .line {
          width: 0.1rem;
          height: 100%;
          position: absolute;
          top: 0;
          left: 50%;
          // z-index: 80;
          background-image: linear-gradient(
            to bottom,
            #0c776a 50%,
            rgba(255, 255, 255, 0) 0
          );
        }

        .arcwrap {
          width: 100%;
          height: 100%;
          position: absolute;
          top: 0;
          left: 0;

          .arc {
            width: inherit;
            height: inherit;
          }
        }

        .textwrap {
          width: 100%;
          height: 100%;
          position: absolute;
          top: 0;
          left: 0;

          span {
            position: absolute;
            top: -3rem;
            left: 48%;
          }
        }

        &:first-child {
          z-index: 99;
        }

        &:last-child {
          border: 1px solid #ddd;
          border-radius: 50%;
        }

        img {
          user-drag: none;
          -webkit-user-drag: none;
          width: inherit;
        }

        .img-frame {
        }

        .img-graph {
          -webkit-touch-callout: none;
          user-select: none;
          opacity: 0.8;
          // z-index: 100;
        }
      }
    }
  }

  .controller {
    width: 20rem;
    margin-left: 5%;
    background-color: #bccee3;
    border-radius: 2rem;

    ul.list {
      height: 100%;
      list-style: none;
      margin: 0;
      padding: 0;
      display: flex;
      flex-flow: column nowrap;
      justify-content: space-between;
      align-items: center;

      li.item {
        height: calc(100% / $itemNum);
        width: 100%;
        box-sizing: border-box;
        display: flex;
        flex-flow: row nowrap;
        justify-content: center;
        align-items: center;
        overflow: hidden;

        img {
          width: 80%;
          user-drag: none;
          -webkit-user-drag: none;
        }
      }

      li.item.active {
        background-image: url("@/assets/images/box.png");
        background-size: 100% 100%;
        object-fit: fill;
      }
    }
  }
}
</style>
