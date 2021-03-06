<template>
  <div
    class="y-alert"
    :class="[{'y-alert-fixed': !htmlRender},'y-alert-' + type, 'y-alert-' + theme]"
    :style="style"
  >
    <div v-show="!hideIcon" class="y-alert-type-icon">
      <slot name="prefix">
        <y-icon v-show="!hideIcon" :type="typeIcons[type]"></y-icon>
      </slot>
    </div>
    <div ref="textWrap" class="y-alert-text-wrapper">
      <div
        ref="text"
        class="y-alert-text"
        :class="{'y-alert-text-scroll-x': scrollParams.direction == 'x', 'y-alert-text-scroll-y': scrollParams.direction == 'y'}"
        :style="textStyle"
      >
        <slot>{{message}}</slot>
      </div>
    </div>
    <div v-show="showClose" class="y-alert-close-icon">
      <slot name="suffix">
        <y-icon type="guanbi" size="12" @click="close"></y-icon>
      </slot>
    </div>
  </div>
</template>

<script>
import { TYPE_ICONS } from "@/util/config";
import componentMixins from "@/mixins/component";
export default {
  name: "y-alert",
  mixins: [componentMixins],
  props: {
    type: {
      type: String,
      default: "text"
    },
    theme: {
      type: String,
      default: "light"
    },
    showClose: {
      type: Boolean,
      default: false
    },
    hideIcon: {
      type: Boolean,
      default: false
    },
    scroll: {
      type: [String, Object],
      default: ""
    }
  },
  data() {
    return {
      // 标识是函数式还是指令式渲染 true代表指令式 即使用<y-alert>标签
      htmlRender: true,
      // 函数式所需的3个参数
      message: "",
      top: 0,
      duration: 0,

      typeIcons: TYPE_ICONS,
      // 帧动画id
      frameId: 0,
      scrollParams: {},
      textWrapRect: {},
      textStyle: {}
    };
  },
  computed: {
    style() {
      return this.htmlRender
        ? ""
        : {
            top: this.top + "px",
            zIndex: this.$YONDUI.getZindex()
          };
    }
  },
  methods: {
    close() {
      if (!this.htmlRender) {
        if (this.onClose && typeof this.onClose === "function") {
          this.onClose(this);
        }
        this.$alert.refresh(this);
      }
      document.body.removeChild(this.$el);
    },
    handleScrollX(x, width) {
      x -= (this.scrollParams.speed * 10) / 1000;
      this.textStyle = {
        transform: `translateX(${x}px)`
      };
      if (width + x <= 0) {
        x = this.textWrapRect.width;
        this.textStyle = {
          transform: `translateX(${x}px)`
        };
      }
      return x;
    },
    handleScrollY(y, height) {
      y -= this.scrollParams.speed;
      this.textStyle.transform = `translateY(${y}px)`;
      if (height + y <= 0) {
        y = 0;
        this.textStyle.transform = `translateY(${y}px)`;
      }
      return y;
    }
  },
  watch: {
    scroll: {
      handler(params) {
        if (params) {
          if (typeof params === "string") {
            this.scrollParams = {
              direction: params,
              speed: params == "x" ? 60 : 19,
              height: 19
            };
          } else {
            this.scrollParams = {
              height: 19,
              speed: params.direction == "x" ? 60 : 1,
              ...params
            };
          }
        }
      },
      immediate: true
    }
  },
  mounted() {
    // 函数式调用 默认3s后关闭
    if (!this.htmlRender && this.duration) {
      setTimeout(() => {
        this.close();
      }, this.duration);
    }

    if (this.scroll) {
      this.textWrapRect = this.$refs["textWrap"].getBoundingClientRect();
      // 横向滚动文字
      if (this.scrollParams.direction === "x") {
        // 由于不能直接获取文本的宽度，所以只能通过计算 文字的长度 * 字体大小 来获取文本的宽度
        // 获取最大值是为了防止出现多行文本的情况
        let list = this.$refs["textWrap"].innerText
          .split("\n")
          .map(text => text.length);
        let textMaxWidth = Math.max(...list) * 14;

        // x 横向移动距离
        let x = 0;
        this.frameId = setInterval(() => {
          x = this.handleScrollX(x, textMaxWidth);
        }, 10);
      }
      // 竖向滚动文字
      else {
        this.textStyle = {
          height: this.scrollParams.height + "px",
          transform: ""
        };
        let height = this.$refs["text"].getBoundingClientRect().height;
        let y = 0;
        this.frameId = setInterval(() => {
          // y 纵向移动距离
          y = this.handleScrollY(y, height);
        }, 1000);
      }
    }
  },
  beforeDestroy() {
    clearInterval(this.frameId);
  }
};
</script>