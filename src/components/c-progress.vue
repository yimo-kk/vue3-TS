<template>
  <div class="c-progress">
    <div
      class="c-progress-outer c-progress-box"
      :style="setProgressBgStyle"
      ref="progress"
    >
      <div class="c-progress-inner" :style="setProgressStyle"></div>
      <div
        v-if="showSlider"
        class="c-progress-slider"
        ref="slider"
        :style="setSliderStyle"
      ></div>
    </div>
    <span v-if="showPerText">{{ tempPercent }}%</span>
  </div>
</template>

<script lang="ts">
import {
  computed,
  defineComponent,
  reactive,
  toRefs,
  PropType,
  onMounted,
} from "vue";
import { prop } from "vue-class-component";
type ColorTable = {
  success: string;
  fail: string;
  warning: string;
  default: string;
};
type ColorTableItem = keyof ColorTable;
const colorTable: ColorTable = {
  success: "#13ce66",
  fail: "#ff4949",
  warning: "#e6a23c",
  default: "#409EFF",
};
export default defineComponent({
  name: "cProgress",
  props: {
    percent: {
      type: Number,
      default: 60,
    },
    showSlider: {
      type: Boolean,
      default: true,
    },
    showPerText: {
      type: Boolean,
      default: true,
    },
    // 进度条的宽度
    width: {
      type: Number,
      default: 300,
    },
    bgColor: {
      type: String,
      default: "#ebeef5",
    },
    progressColor: {
      type: String,
      default: "#409EFF",
    },
    // 滑块的宽度
    sliderWidth: {
      type: Number,
      default: 20,
    },
    // 颜色的类型
    type: {
      type: String as PropType<ColorTableItem>,
      default: "default",
    },
  },
  setup(props) {
    const data = reactive({
      sliderLeft: 0, // 滑块相对父元素发x坐标
      progressWidth: 0, // 进度条当前的的宽度
      tempPercent: 0,
    });
    const setProgressBgStyle = computed(() => {
      return {
        // 加上滑块的宽度
        width: `${props.width + props.sliderWidth}px`,
      };
    });
    const setSliderStyle = computed(() => {
      const color = colorTable[props.type] || props.progressColor;
      return {
        border: `1px solid ${color}`,
        width: `${props.sliderWidth}px`,
        height: `${props.sliderWidth}px`,
        left: `${data.sliderLeft}px`,
      };
    });
    const setProgressStyle = computed(() => {
      const color = colorTable[props.type] || props.progressColor;
      return {
        width: `${data.progressWidth}px`,
        background: color,
      };
    });
    const addListener = () => {
      const slider: HTMLDivElement | null =
        document.querySelector(".c-progress-slider");
      const progress: HTMLDivElement | null =
        document.querySelector(".c-progress-box");
      let isClickSlider = false;
      let distance = 0; // 滑块与点击坐标的绝对距离
      (progress as HTMLDivElement).onclick = (e) => {
        // 阻止事件冒泡
        if (e.target == slider) {
          return;
        }
        let curX = (progress as HTMLDivElement).offsetLeft;
        data.sliderLeft = e.offsetX - curX;
        if (data.sliderLeft <= 0) {
          data.sliderLeft = 0;
        }
        if (data.sliderLeft >= props.width) {
          data.sliderLeft = props.width;
        }
        _countCurPercent();
      };
      (slider as HTMLDivElement).onmousedown = (e) => {
        isClickSlider = true;
        let curX = (slider as HTMLDivElement).offsetLeft;
        distance = e.pageX - curX; // 得出绝对距离
      };
      (progress as HTMLDivElement).onmousemove = (e) => {
        if (isClickSlider) {
          // 判断是否已经超出进度条的长度
          if (
            e.pageX - distance >= 0 &&
            e.pageX - distance <= props.width - 0
          ) {
            data.sliderLeft = e.pageX - distance;
            _countCurPercent();
          }
        }
      };
      (progress as HTMLDivElement).onmouseup = () => {
        isClickSlider = false;
      };
    };
    onMounted(() => {
      data.sliderLeft = (props.width / 100) * props.percent;
      data.progressWidth = data.sliderLeft + props.sliderWidth; // 滑块的x坐标加上滑块的宽度
      data.tempPercent = props.percent;
      addListener();
    });
    // 算出百分比
    const _countCurPercent = () => {
      data.tempPercent = Math.ceil((data.sliderLeft / props.width) * 100);
      data.progressWidth = data.sliderLeft + 20;
      // 取整的时候宽度可能不为0，所以在0和100的时候也将宽度取整
      if (data.tempPercent <= 0) {
        data.progressWidth = 0;
        data.sliderLeft = 0;
      }
      if (data.tempPercent >= 100) {
        data.progressWidth = props.width + 20;
        data.sliderLeft = props.width;
      }
      // this.$emit('percentChange', this.tempPercent)
    };
    const refData = toRefs(data);
    return {
      setProgressBgStyle,
      setSliderStyle,
      ...refData,
      setProgressStyle,
    };
  },
});
</script>

<style lang="scss" scoped>
  .c-progress {
    $width: 300px;
    $radius: 5px;
    display: flex;
    align-items: center;
    
    span {
      margin-left: 5px;
      font-size: 14px;
      color: #666;
    }
    
    .c-progress-outer {
      width: $width;
      height: 10px;
      border-radius: $radius;
      background: #ebeef5;
      position: relative;
      display: flex;
      align-items: center;
      
      .c-progress-inner {
        width: 100px;
        height: 10px;
        background: #409EFF;
        border-radius: $radius;
      }
      
      .c-progress-slider {
        width: 20px;
        height: 20px;
        border-radius: 50%;
        background: #fff;
        border: 1px solid #409EFF;
        position: absolute;
        z-index: 10;
        left: 10px;
      }
    }
  }
</style>
