<template>
  <div
    @click="$emit('pressed')"
    :style="{
      width: circle.size + 'px',
      height: circle.size + 'px',
      top: circle.x + 'px',
      left: circle.y + 'px',
    }"
    class="rounded-full bg-white z-20 absolute"
  ></div>
</template>

<script>
export default {
  props: {
    circle: {
      required: true,
      type: Object,
    },
  },
};
</script>
