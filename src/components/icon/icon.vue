<template>
  <i
    @click="handleClick"
    :class="`iconfont y-icon-${type}`"
    :style="{'font-size': size + 'px', 'color': color}"
  ></i>
</template>

<script>
export default {
  name: "y-icon",
  props: {
    type: {
      type: String,
      required: true
    },
    size: {
      type: String,
      default: ""
    },
    color: {
      type: String,
      default: ""
    }
  },
  methods: {
    handleClick(e) {
      this.$emit("click", e);
    }
  }
};
</script>