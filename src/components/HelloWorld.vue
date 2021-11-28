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
import { Options, Vue } from "vue-class-component";
import { Component, Prop, Watch, Emit } from "vue-property-decorator";
const colorTable = {
  success: "#13ce66",
  fail: "#ff4949",
  warning: "#e6a23c",
  default: "#409EFF",
};
@Options({
  props: {
    msg: String,
    percent: Number,
    showSlider: Boolean,
    showPerText: Boolean,
    // 进度条的宽度
    width: Number,
    bgColor: String,
    progressColor: String,
    // 滑块的宽度
    sliderWidth: Number,
    // 颜色的类型
    type: String,
  },
})
export default class HelloWorld extends Vue {
  msg: string|undefined;
  @Prop({ default: 60 }) percent?: Number;
  @Prop({ default: true }) showSlider!: boolean;
  @Prop({ default: true }) showPerText!: boolean;
  @Prop({ default: 300 }) width!: Number;
  @Prop({ default: "#ebeef5" }) bgColor!: string;
  @Prop({ default: "#409EFF" }) progressColor!: string;
  @Prop({ default: 20 }) sliderWidth!: Number;
  @Prop({ default: colorTable.default }) type!: string;

  sliderLeft = 0; // 滑块相对父元素发x坐标
  progressWidth = 0; // 进度条当前的的宽度
  tempPercent = 0;
  distance = 0; // 滑块与点击坐标的绝对距离
  // 设置进度条的背景样式
  get setProgressBgStyle(): any {
    return {
      // 加上滑块的宽度
      width: `${this.width + this.sliderWidth}px`,
    };
  }
  // 设置进度条的样式
  get setProgressStyle(): any {
    const color = colorTable[this.type] || this.progressColor;
    return {
      width: `${this.progressWidth}px`,
      background: color,
    };
  }
  // 设置滑块的样式
  get setSliderStyle(): any {
    const color = colorTable[this.type] || this.progressColor;
    return {
      border: `1px solid ${color}`,
      width: `${this.sliderWidth}px`,
      height: `${this.sliderWidth}px`,
      left: `${this.sliderLeft}px`,
    };
  }

  mounted(): void {
    this.sliderLeft = (this.width / 100) * this.percent;
    this.progressWidth = this.sliderLeft + this.sliderWidth; // 滑块的x坐标加上滑块的宽度
    this.tempPercent = this.percent;
    this.addListener();
  }
  addListener() {
    const slider: HTMLDivElement | null =
      document.querySelector(".c-progress-slider");
    const progress: HTMLDivElement | null =
      document.querySelector(".c-progress-box");
    let isClickSlider = (false(progress as HTMLDivElement).onclick = (e) => {
      // 阻止事件冒泡
      if (e.target == slider) {
        return;
      }
      let curX = (progress as HTMLDivElement).offsetLeft;
      this.sliderLeft = e.offsetX - curX;
      if (this.sliderLeft <= 0) {
        this.sliderLeft = 0;
      }
      if (this.sliderLeft >= this.width) {
        this.sliderLeft = this.width;
      }
      this._countCurPercent();
    });
    (slider as HTMLDivElement).onmousedown = (e) => {
      isClickSlider = true;
      let curX = slider.offsetLeft;
      distance = e.pageX - curX; // 得出绝对距离
    };
    (progress as HTMLDivElement).onmousemove = (e) => {
      if (isClickSlider) {
        // 判断是否已经超出进度条的长度
        if (e.pageX - distance >= 0 && e.pageX - distance <= this.width - 0) {
          this.sliderLeft = e.pageX - distance;
          this._countCurPercent();
        }
      }
    };
    (progress as HTMLDivElement).onmouseup = () => {
      isClickSlider = false;
    };
  }
  // 算出百分比
  _countCurPercent() {
    this.tempPercent = Math.ceil(
      parseInt((this.sliderLeft / this.width) * 100)
    );
    this.progressWidth = this.sliderLeft + 20;
    // 取整的时候宽度可能不为0，所以在0和100的时候也将宽度取整
    if (this.tempPercent <= 0) {
      this.progressWidth = 0;
      this.sliderLeft = 0;
    }
    if (this.tempPercent >= 100) {
      this.progressWidth = this.width + 20;
      this.sliderLeft = this.width;
    }
    this.$emit("percentChange", this.tempPercent);
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>

function setProgressStyle() {
  throw new Error("Function not implemented.");
}
