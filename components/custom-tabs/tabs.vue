<template>
  <view class="tabs">
    <view class="tab" :class="syncActiveIndex === index ? 'tab--active' : ''" v-for="(item, index) in tabs" :key="index" @click="selectTab(index)">
      <span>{{ item.name }}</span>
      <view class="img-box tab__arrow" v-if="item.showArrow" :style="{ transform: 'rotate(' + towards[item.arrowTowards || 'right'] + 'deg)' }">
        <image src="../../static/images/components/tab_arrow.png" mode=""></image>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  props: {
    activeIndex: Number, // 当前选中项
    tabs: {
      // 选项
      type: Array,
      default: [
        {
          name: '标签',
          showArrow: true, // 显示右箭头
          arrowTowards: 'right', // 箭头朝向
          customClick: true // 自定义点击效果
        }
      ]
    }
  },
  data() {
    return {
      towards: { top: -90, right: 0, bottom: 90, left: 180 },
      syncActiveIndex: 0
    };
  },
  mounted() {
    this.syncActiveIndex = this.activeIndex;
  },
  methods: {
    selectTab(index) {
      this.$emit('onselect', { index, item: this.tabs[index] });
      if (this.tabs[index].customClick) return false;
      this.$emit('update:activeIndex', index);
      this.syncActiveIndex = index;
    }
  }
};
</script>

<style scoped>
.tabs {
  height: 100rpx;
  background-color: white;
  box-shadow: 0 4rpx 8rpx 2rpx rgba(0, 0, 0, 0.1);
  border-top: 2rpx solid #f9f9f9;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 64rpx;
  box-sizing: border-box;
}

.tab {
  font-size: var(--theme--font-size-s);
  position: relative;
  height: 40rpx;
  line-height: 40rpx;
  display: flex;
  align-items: center;
}

.tab::after {
  content: '';
  width: 0%;
  position: absolute;
  height: 4rpx;
  border-radius: 2rpx;
  background-color: var(--theme--main-color);
  left: 100%;
  bottom: -4rpx;
  transition: all 0.2s linear;
}

.tab--active::after {
  width: 100%;
  left: 0;
}

.tab--active ~ .tab::after {
  left: 0;
}

.tab__arrow {
  width: 20rpx;
  height: 20rpx;
  margin-left: 10rpx;
}
</style>
