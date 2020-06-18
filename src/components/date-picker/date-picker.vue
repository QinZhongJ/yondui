<template>
  <div class="y-date-picker" :style="{width}">
    <y-input
      v-model="inputValue"
      type="textarea"
      readonly
      :disabled="disabled"
      :size="size"
      :clearable="clearable"
      :placeholder="placeholder"
      @on-focus="handleInputFocus"
      @on-clear="handleClear"
    >
      <template slot="prefix">
        <slot name="prefix"></slot>
      </template>
      <y-icon slot="suffix" size="14" color="#c0c4cc" type="calendar"></y-icon>
    </y-input>
  </div>
</template>

<script>
import Vue from "vue";
import DatePcikerDropdown from "./date-picker-dropdown.vue";
import { setDateFormat } from "@/util/tools";
if (!new Date().Format) {
  setDateFormat(); // 注册格式化时间函数
}
export default {
  name: "y-date-picker",
  props: {
    value: [Date, String, Number, Array],
    width: String,
    size: String,
    type: {
      type: String,
      default: "date"
    },
    placeholder: {
      type: String,
      default: "请选择"
    },
    format: String,
    clearable: Boolean,
    disabled: Boolean,
    multiple: Boolean,
    disabledDate: Function,
    filterTime: Function
  },
  data() {
    return {
      // input框展示内容
      inputValue: "",
      // 给dropdown的值
      dateValue: null,
      dropdownVm: null,
      formats: {
        year: "yyyy",
        yearrange: "yyyy",
        month: "yyyy-MM",
        monthrange: "yyyy-MM",
        date: "yyyy-MM-dd",
        daterange: "yyyy-MM-dd",
        datetime: "yyyy-MM-dd hh:mm:ss",
        datetimerange: "yyyy-MM-dd hh:mm:ss"
      }
    };
  },
  computed: {
    // 时间格式
    dateFormat() {
      return this.formats[this.type];
    }
  },
  methods: {
    emitChange(date, time) {
      // 范围选择
      if (this.type.indexOf("range") != -1) {
        let startString = this.handleFormat(date[0], time ? time[0] : "");
        let endString = this.handleFormat(date[1], time ? time[1] : "");
        let formatString = startString.format + " 至 " + endString.format;
        this.inputValue = formatString;
        this.$emit("input", [startString.normal, endString.normal]);
        this.$emit("on-change", [startString.normal, endString.normal]);
        return;
      }

      // 当为多选时
      if (this.multiple) {
        let list = this.getMultipleList(date, time);
        this.inputValue = list.map(item => item.format).toString();
        this.$emit(
          "input",
          list.map(item => item.normal)
        );
        this.$emit(
          "on-change",
          list.map(item => item.normal)
        );
        return;
      }

      let formatString = this.handleFormat(date, time);
      this.inputValue = formatString.format;
      this.$emit("input", formatString.normal);
      this.$emit("on-change", formatString.normal);
    },
    handleFormat(date, time = "") {
      let datetime = new Date(date).Format("yyyy-MM-dd") + " " + time;
      return {
        // 显示格式
        format: new Date(datetime).Format(this.format || this.dateFormat),
        // 实际格式
        normal: new Date(datetime).Format(this.dateFormat)
      };
    },
    handleInputFocus() {
      let pickerRect = this.$el.getBoundingClientRect();
      if (!this.dropdownVm) {
        let _this = this;
        const DropdownConstructor = Vue.extend(DatePcikerDropdown);
        this.dropdownVm = new DropdownConstructor({
          data: {
            top: pickerRect.bottom + 5,
            left: pickerRect.left,
            parentVm: _this,
            parentEl: _this.$el
          }
        }).$mount();
        document.body.appendChild(this.dropdownVm.$el);
        this.dropdownVm.show();
      } else {
        this.dropdownVm.top = pickerRect.bottom + 5;
        this.dropdownVm.left = pickerRect.left;
        this.dropdownVm.show();
      }
    },
    // 点击清空按钮
    handleClear() {
      this.$emit("on-clear");
      this.inputValue = "";
    },
    getMultipleList(dateList, timeList) {
      let list = [];
      for (let i in dateList) {
        list.push(this.handleFormat(dateList[i], timeList ? timeList[i] : ""));
      }
      return list;
    }
  },
  watch: {
    value: {
      handler(value) {
        if (value) {
          if (!this.multiple && Array.isArray(value)) {
            let ifBigger = true;
            if (value[0] && value[1]) {
              // 判断谁大谁小
              ifBigger =
                new Date(value[1]).getTime() > new Date(value[0]).getTime();
            }
            let firstDate = value[ifBigger ? 0 : 1];
            let secondDate = value[ifBigger ? 1 : 0];
            this.dateValue = [firstDate, secondDate];
            let text = "";
            if (firstDate) {
              text =
                new Date(firstDate).Format(this.format || this.dateFormat) +
                "-";
            }
            if (secondDate) {
              text += new Date(secondDate).Format(
                this.format || this.dateFormat
              );
            }
            this.inputValue = text;
          } else {
            this.dateValue = value;
            if (this.multiple) {
              this.inputValue = this.getMultipleList(value)
                .map(item => item.format)
                .toString();
            } else {
              this.inputValue = new Date(value).Format(
                this.format || this.dateFormat
              );
            }
          }
        } else {
          this.dateValue = "";
          this.inputValue = "";
        }
      },
      immediate: true
    }
  }
};
</script>