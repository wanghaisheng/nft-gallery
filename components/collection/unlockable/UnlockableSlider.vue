<template>
  <div class="unlockable-slider border rounded-[2.5rem]">
    <div
      v-if="value > 0"
      class="unlockable-slider-inner border rounded-[2.5rem]"
      :style="widthStyle"></div>
  </div>
</template>

<script lang="ts" setup>
const props = withDefaults(
  defineProps<{
    value: number
  }>(),
  {
    value: 0,
  },
)

const widthStyle = computed(() => ({ width: `${props.value * 100}%` }))
</script>

<style lang="scss" scoped>
@import '@/assets/styles/abstracts/variables.scss';
.unlockable-slider {
  position: relative;
  height: 14px;
  overflow: hidden;
  @include ktheme() {
    background: theme('background-color');
  }
  &-inner {
    position: absolute;
    left: -1px;
    top: -1px;
    height: 100%;
    background: $primary;
    box-sizing: content-box;
  }
}
</style>
