<template>
  <div class="window-action">
    <span class="btn small" @click="handleWindowChange(isMin)">
      <LsIcon icon="navi_zxh_icon" render-svg></LsIcon>
    </span>
    <span class="btn big" :class='isLogin && "disable"' @click="handleWindowChange(isMax)">
      <LsIcon icon="navi_sx_icon" render-svg></LsIcon>
    </span>
    <span class="btn close" @click="handleWindowChange(isClose)">
      <LsIcon icon="navi_gb_icon" render-svg></LsIcon>
    </span>
  </div>
</template>

<script>
import { renderProcess } from '@lanshu/render-process';
import LsIcon from './ls-icon';

export default {
  name: 'Window-action',
  props: {
    isLogin: {
      type: Boolean,
      default: false
    }
  },
  components: {
    LsIcon
  },
  data() {
    return {
      isMin: 'min',
      isMax: 'max',
      isClose: 'close',
    }
  },
  methods: {
    handleWindowChange(type) {
      console.log(type);
      if (type === this.isMax && this.isLogin) return;
      renderProcess.changeWindow(type);
    },
  },
};
</script>

<style scoped lang="scss">
.window-action {
  display: flex;
  justify-content: flex-end;
  align-items: center;

  .btn {
    width: 20px;
    height: 20px;
    margin-left: 10px;
    border-radius: 50%;
    cursor: pointer;

    &.disable {
      cursor: not-allowed;
      opacity: .5;
    }
  }
}
</style>
