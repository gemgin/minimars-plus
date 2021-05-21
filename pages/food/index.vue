<template lang="pug">
view.orderFood_box
  view.orderFood_left
    scroll-view.orderFood_left_scroll(scroll-y="true", show-scrollbar="true")
      view.orderFood_left_title(
        v-for="(item, index) in tabBars",
        :key="index",
        @click="setid(item.uid)",
        :class="item.uid == change ? 'active' : ''"
      )
        span {{ item.name }}
  view.orderFood_right
    // 轮播
    view.orderFood_right_banner
      swiper.swiper(
        :autoplay="swiperAutoplay",
        :interval="3000",
        :duration="1000",
        :current="swiperCurrent",
        indicator-dots="true",
        circular="true",
        indicator-color="#B9B9B9",
        indicator-active-color="#9B9B9B"
      )
        swiper-item(v-for="item in bannerPosts", :key="item.id")
          img.swiper_item(:src="item.posterUrl", mode="aspectFill")
    // 特色推荐
    view.orderFood_right_content_box
      scroll-view.orderFood_right_content_scroll(
        :scroll-y="true",
        :scroll-into-view="clickId",
        @scroll="asideScroll",
        @scrolltolower="scrolltolower",
        @scrolltoupper="scrolltoupper"
      )
        // :lower-threshold="200" :upper-threshold="200" :scroll-with-animation="true"
        view.orderFood_right_content
          view(
            v-for="(item, index) in tabBars",
            :key="index",
            :id="'po' + item.uid"
          )
            text.name.orderFood_right_title {{ item.name }}
            view.orderFood_right_title_content(
              v-for="(item2, index2) in item.products",
              :key="index2"
            )
              view.orderFood_right_title_content-left(
                @click="open(index, index2, item2)"
              )
                img(:src="item2.imageUrl")
              view.orderFood_right_title_content-right
                view.right_title_content-right-title {{ item2.name }}
                view.right_title_content-right-detail(
                  @click="open(index, index2, item2)"
                ) {{ item2.description }}
                view.right_title_content-right-money_box
                  view.right_title_content-right-money_box_money
                    | rmb {{ item2.buyPrice }}
                  view.right_title_content-right-money_box_add(
                    v-if="item2.numbers == 0"
                  )
                    view
                    view
                    view(@click="foodAdd(item2)")
                      img.foodAdd_img(src="../../static/images/add_deputy.png")
                  view.right_title_content-right-money_box_add(
                    v-if="item2.numbers > 0"
                  )
                    view(@click="foodreduce(item2)")
                      img.foodreduce_img(src="../../static/images/minus_deputy.png")
                    view {{ item2.numbers }}
                    view(@click="foodAdd(item2)")
                      img.foodAdd_img(src="../../static/images/add_deputy.png")
      // 详情
      uni-popup(ref="popup", type="center", :tabbar="true")
        view.gift_box
          view.gift_box_top
            img(:src="item.imageUrl", mode="aspectFill")
            view.gift_box_top-close
              img.gift_box_clear_right(
                src="../../static/images/clear.png",
                @click="close()"
              )
          view.gift_box_top_content
            view.gift_box_top_content-title {{ item.name }}
            view.gift_box_top_content-box {{ item.description }}
          view.gift_box_top_footer
            view.gift_box_top_footer-left
              | rmb {{ item.buyPrice }}
            view.gift_box_top_footer-right(@click="unpopAdd(index)")
              img(src="../../static/images/add_deputy.png")
      // 加购选择
      view.orderFood_choose(v-if="settlement == 1")
        view.orderFood_choose-left
          img.orderFood_choose-left_img(
            src="../../static/images/orderFood/shopcar.png"
          )
          view.orderFood_choose-left-line
          view.orderFood_choose-left_money
            | rmb {{ playMoney }}
        view.orderFood_choose-right(@click="goChoose()")
          view.orderFood_choose-right-choose 选好了
          img(src="../../static/images/orderFood/white_right.png")
</template>

<script>
import uniPopup from '@/components/uni-popup/uni-popup.vue';
import foodMenu from '@/components/food-menu.vue';
import { sync } from 'vuex-pathify';

export default {
  components: {
    uniPopup,
    foodMenu
  },
  data() {
    return {
      isScanning: false,
      // 购物车价钱
      playMoney: 0,
      change: 0,
      swiperHeight: 0,
      swiperAutoplay: true,
      menuWidth: 0,
      foodWidth: 0,
      tabIndexShow: 0,
      tabIndex: 0,
      bannerPosts: [],
      current: 0,
      swiperCurrent: 0,
      // 购物车显示隐藏
      settlement: 0,
      tabBars: [],
      clickId: '',
      change: 0,
      topList: [],
      isLeftClick: false,
      // 弹框得到索引
      unpooIndex: '',
      unpooIndex2: '',
      item: {},
      storeCode: '',
      storeId: '',
      tableId: ''
    };
  },
  onLoad(option) {
    console.log('food/index:onLoad', option);
    this.getbanner();
    if (option.s && option.t) {
      this.storeCode = option.s;
      this.tableId = option.t;
    }
  },
  async onShow() {
    console.log("food/index:onShow");
    if (this.isScanning) return;
    try {
      this.isScanning = true;
      await this.scanTableCode();
      this.isScanning = false;
      uni.showLoading();
      await this.getMenu();
      uni.hideLoading();
    } catch (e) {
      uni.showToast({ title: e, icon: "none" });
    }
  },
  onHide() {
    this.swiperAutoplay = false;
  },
  methods: {
    scanTableCode() {
      return new Promise((resolve, reject) => {
        uni.scanCode({
          success: (res) => {
            // console.log(res.scanType, res.path, res.result);
            if (res.path) {
              const [, s, t] = res.path.match(/s=(.*)&t=(.*)/);
              console.log(s, t);
              if (!s || !t) {
                return reject("无效点餐二维码");
              }
              this.storeCode = s;
              this.tableId = t;
              resolve();
            } else {
              reject("无效点餐二维码");
            }
          },
          fail: function (res) {
            reject("扫码失败");
            uni.switchTab({
              url: "../index/index",
            });
          },
        });
      });
    },
    async getMenu() {
      const storeMenu = await this.$axios.getRequest('/store-menu', {
        storeCode: this.storeCode,
        tableId: this.tableId
      });
      storeMenu.menu.forEach(i => {
        i.products.forEach(j => {
          j.numbers = 0;
        });
      });
      this.tabBars = storeMenu.menu;
      this.tableId = storeMenu.tableId;
      this.storeId = storeMenu.store.id;
    },
    async getbanner() {
      this.bannerPosts = await this.$axios.getRequest('/post', {
        tag: 'food'
      });
    },
    // 轮播
    changeSwiper(e) {
      this.swiperCurrent = e.detail.current;
    },
    // 弹框打开,关闭
    open(index, index2, item) {
      this.item = item;
      this.unpooIndex = index;
      this.unpooIndex2 = index2;
      this.$refs.popup.open();
      wx.hideTabBar();
    },
    close() {
      this.$refs.popup.close();
      wx.showTabBar();
    },
    // 弹框加
    unpopAdd() {
      this.tabBars.forEach((item, k) => {
        if (k == this.unpooIndex) {
          item.products.forEach((j, l) => {
            if (l == this.unpooIndex2) {
              j.numbers++;
            }
          });
        }
      });
      this.settlement = 1;
      //总价格
      this.playMoney = 0;
      this.tabBars.forEach(item => {
        item.products.forEach(j => {
          this.playMoney += j.numbers * j.buyPrice;
        });
      });
      this.$refs.popup.close();
      wx.showTabBar();
    },
    // 购物车数量加减
    foodreduce(item2) {
      item2.numbers--;
      this.playMoney = this.playMoney - item2.buyPrice;
      if (this.playMoney == 0) {
        this.settlement = 0;
      }
    },
    foodAdd(item2) {
      for (let i = 0; i < this.tabBars.length; i++) {
        for (let j = 0; j < this.tabBars[i].products.length; j++) {
          if (item2.numbers == 0) {
            this.settlement = 1;
            break;
          }
        }
      }
      //count
      item2.numbers++;
      //总价格
      this.playMoney = 0;
      this.tabBars.forEach(item => {
        item.products.forEach(j => {
          this.playMoney += j.numbers * j.buyPrice;
        });
      });
    },

    goChoose() {
      let selectshop = []; //选中的餐品
      this.tabBars.forEach(item => {
        item.products.forEach(items => {
          if (items.numbers > 0) {
            selectshop.push(items);
          }
        });
      });
      console.log(selectshop);
      uni.navigateTo({
        url: './order?selectshop=' + JSON.stringify(selectshop) + '&tableId=' + this.tableId + '&storeId=' + this.storeId
      });
    },
    // 鼠标点击
    setid(index) {
      this.clickId = 'po' + index;
      this.change = index;
      this.isLeftClick = true;
    },

    // 滚动到底部
    scrolltolower() {
      this.change = this.tabBars[this.tabBars.length - 1].uid;
    },
    // 滚动到顶部
    scrolltoupper() {
      this.change = this.tabBars[0].uid;
    },
    //右侧栏滚动
    asideScroll(e) {
      if (!this.isLeftClick) {
        this.calcSize();
      }
      let scrollTop = e.detail.scrollTop;
      let tabs = this.tabBars.filter(item => item.top <= scrollTop).reverse();
      if (tabs.length > 0) {
        this.change = tabs[0].uid;
      }
    },
    //计算右侧栏每个tab的高度等信息
    calcSize() {
      let h = 0;
      this.tabBars.forEach(item => {
        let view = uni.createSelectorQuery().select('#po' + item.uid);
        view
          .fields(
            {
              size: true
            },
            data => {
              item.top = h;
              h += data.height;
              item.bottom = h;
            }
          )
          .exec();
      });
      this.isLeftClick = true;
    }
  }
};
</script>

<style lang="less" scoped>
.orderFood_box {
  display: flex;

  .orderFood_left {
    width: 132rpx;
    // background: #FFFFFF;
    background: #f7f7f7;
    height: 100vh;

    .orderFood_left_scroll {
      height: 100vh;

      .active {
        background-color: #ffffff;
        border-left: 7rpx solid #9fcdff;
      }

      .orderFood_left_title {
        width: 90rpx;
        height: 100rpx;
        border-bottom: 1px solid #d1d1d1;
        line-height: 25rpx;
        margin: 0 auto;
        display: flex;
        justify-content: center;
        align-items: center;

        span {
          font-size: var(--theme--font-size-s);
          font-weight: var(--theme--font-weight-light);
          white-space: wrap;
        }
      }
    }
  }

  // 右边
  .orderFood_right {
    width: 620rpx;
    // height: 100vh;

    // 轮播
    .orderFood_right_banner {
      width: 100%;
      height: 270rpx;
      background: #f0f0f0;

      .swiper {
        width: 100%;
        height: 100%;
        border-radius: var(--theme--border-radius);

        .swiper_item {
          width: 554rpx;
          // height: 256rpx;
          height: 22.5vh;
          border-radius: 20rpx;

          image {
            width: 554rpx;
            height: 256rpx;
            // border-radius: 20rpx;
          }
        }
      }
    }

    .orderFood_right_content_box {
      width: 554rpx;
      height: calc(100vh - 270rpx);
      margin: 0 auto;

      .orderFood_right_content_scroll {
        height: 100%;

        .orderFood_right_content {
          margin-top: 20rpx;
          padding-bottom: 90rpx;

          .name {
            margin: 40rpx 0;
            display: block;
            font-size: var(--theme--font-size-s);
            color: var(--theme--font-main-color);
          }

          .orderFood_right_title_content {
            display: flex;
            justify-content: space-between;
            margin-bottom: 20rpx;
            margin-top: 20rpx;
            min-height: 168rpx;
            position: relative;

            .orderFood_right_title_content-left {
              width: 160rpx;
              height: 160rpx;
              border-radius: 20rpx;

              image {
                width: 160rpx;
                height: 160rpx;
                border-radius: 20rpx;
              }
            }

            .orderFood_right_title_content-right {
              width: 360rpx;
              position: relative;
              min-height: 175rpx;
              .right_title_content-right-title {
                width: 100%;
                height: 36rpx;
                font-size: var(--theme--font-size-m);
                line-height: 36rpx;
                margin-bottom: 5rpx;
              }

              .right_title_content-right-detail {
                width: 360rpx;
                height: 56rpx;
                font-size: var(--theme--font-size-s);
                color: var(--theme--font-deputy-color);
                font-weight: var(--theme--font-weight-light);
                line-height: 28rpx;
                overflow: hidden;
              }

              .right_title_content-right-money_box {
                width: 360rpx;
                display: flex;
                justify-content: space-between;
                align-items: center;
                // margin-top: 30rpx;
                position: absolute;
                bottom: 0;
                .right_title_content-right-money_box_money {
                  height: 42rpx;
                  font-size: var(--theme--font-size-m);
                }

                .right_title_content-right-money_box_add {
                  display: flex;
                  justify-content: space-between;
                  align-items: center;
                  width: 120rpx;
                  height: 40rpx;
                  line-height: 26rpx;
                  font-size: var(--theme--font-size-m);

                  .foodreduce_img {
                    width: 37rpx;
                    height: 37rpx;
                  }

                  .foodAdd_img {
                    width: 40rpx;
                    height: 40rpx;
                  }
                }
              }
            }
          }
        }
      }

      // 食物详情弹框
      .gift_box {
        width: 640rpx;
        height: 952rpx;
        background: #ffffff;
        border-radius: var(--theme--border-radius);

        .gift_box_top {
          position: relative;

          image {
            width: 640rpx;
            height: 640rpx;
            border-radius: 20rpx;
          }

          .gift_box_top-close {
            position: absolute;
            top: 36rpx;
            right: 36rpx;

            image {
              width: 40rpx;
              height: 40rpx;
              background: #666666;
              opacity: 0.8;
            }
          }
        }

        .gift_box_top_content {
          margin-left: 25rpx;

          .gift_box_top_content-title {
            width: 100%;
            height: 36rpx;
            font-size: var(--theme--font-size-m);
            color: var(--theme--font-main-color);
            line-height: 36rpx;
            margin: 15rpx 0 10rpx 0;
          }

          .gift_box_top_content-box {
            height: 56rpx;
            font-size: var(--theme--font-size-s);
            color: var(--theme--font-main-color);
            line-height: 42rpx;
          }
        }

        .gift_box_top_footer {
          margin-top: 95rpx;
          width: 640rpx;
          height: 90rpx;
          background: #f7f7f7;
          border-radius: 20rpx;
          display: flex;
          justify-content: space-between;
          align-items: center;

          .gift_box_top_footer-left {
            width: 88rpx;
            height: 50rpx;
            font-size: var(--theme--font-size-m);
            color: var(--theme--bg-main-color);
            line-height: 65rpx;
            margin-left: 10rpx;
          }

          .gift_box_top_footer-right {
            margin-right: 10rpx;
            margin-top: 30rpx;

            image {
              width: 40rpx;
              height: 40rpx;
            }
          }
        }
      }
    }
  }

  // 选择商品
  .orderFood_choose {
    z-index: 1;
    position: fixed;
    bottom: 20rpx;
    width: 594rpx;
    height: 102rpx;
    background: var(--theme--deputy-color);
    box-shadow: var(--theme--box-shadow);
    border-radius: var(--theme--border-radius);
    left: 50%;
    transform: translateX(-50%);
    padding: 0 36rpx;
    display: flex;
    justify-content: space-between;

    .orderFood_choose-left {
      display: flex;
      align-items: center;
      justify-content: space-between;

      .orderFood_choose-left_img {
        width: 40rpx;
        height: 40rpx;
        z-index: 1;
        margin-right: 30rpx;
      }

      .orderFood_choose-left-line {
        width: 2rpx;
        height: 32rpx;
        background-color: #ffffff;
      }

      .orderFood_choose-left_money {
        height: 42rpx;
        font-size: var(--theme--font-size-m);
        color: #ffffff;
        line-height: 45rpx;
        margin-left: 26rpx;
      }
    }

    .orderFood_choose-right {
      display: flex;
      align-items: center;
      margin-right: 15rpx;
      justify-content: space-around;

      .orderFood_choose-right-choose {
        height: 42rpx;
        font-size: var(--theme--font-size-m);
        color: #ffffff;
        line-height: 42rpx;
        margin-right: 11rpx;
      }

      image {
        width: 17rpx;
        height: 18rpx;
      }
    }
  }
}
</style>
