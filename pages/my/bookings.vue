<template lang="pug">
.orderLlist
  div
    custom-tabs(:tabs="tabs",:activeIndex.sync="active",@onselect="changeTab")
    div(style="width: 100%")
    // 已点餐
    view.one(v-show="active == 0")
      view.haveOrder_box(v-if="QRCodeHide")
        // scan a QR code
        view.haveOrder_scan_box
          view.haveOrder_scan
            view.haveOrder_scan-code
            view.haveOrder_scan_content 
              | 您还没有点餐
              br
              | 快去点点东西吧
            view.haveOrder_scan_btn
              | 去点单 ORDER 
      view.accomplish_border(v-if="QRCodeHide == false")
        
        view.order(v-for="(i, t) in ReservedOrders", :key="t")
          view.order-title
            view.order__date {{ i.updatedAt }}
            view.order__status {{ i.status }}
          view.order-content
            view.order__shop {{ i.store.name }}
            view.order__list
              view.order__goods
                view.img-box.order__goods__item(
                      v-for="(item, index) in i.items.slice(0,4)",
                      :key="index"
                    )
                  img(:src="item.productImageUrl")
              view.order__total
                view.order__total__num 共{{ i.items.length }}件
                view.order__total__price rmb 0
          
    // 已预约
    view.one(v-show="active == 1")
      view.order(
        v-for="(item, index) in ReservedOrders",
        :key="index"
      )
        view.order-title
          view.order__date {{ item.date }}
          view.order__status {{ item.status }}
        view.order-content
          view.order__shop {{ item.title }}
          view.order__text {{ item.kidsCount }}儿童; {{ item.adultsCount }}成人
          
    // 已完成
    view.one(v-show="active == 2")
      view.accomplish_border
      
        view.order(v-for="(i, t) in ReservedOrders", :key="t")
          view.order-title
            view.order__date {{ i.updatedAt }}
            view.order__status {{ i.status }}
          view.order-content
            view.order__shop {{ i.store.name }}
            view.order__list
              view.order__goods
                view.img-box.order__goods__item(
                      v-for="(item, index) in i.items.slice(0,4)",
                      :key="index"
                    )
                  img(:src="item.productImageUrl")
              view.order__total
                view.order__total__num 共{{ i.items.length }}件
                view.order__total__price rmb 0
        // two
        //
          <view class="accomplish_box">
          <view class="accomplish_top">
          <view class="accomplish_top_title_box">
          <view class="accomplish_top_title">
          <view class="accomplish_top_titlename">
          长宁天山店
          </view>
          <span class="accomplish_top_date">2020-12-18 18:24:29</span>
          </view>
          <view class="accomplish_top_btn_box">
          已完成
          <image src="../../static/images/111.png" class="accomplish_top_btn" />
          </view>
          </view>
          </view>
          <view class="accomplish_contentbox" @click="open()">
          <view class="accomplish_contentbox_left">
          <scroll-view scroll-x="true" class="modeOf_Payment-box">
          <view class="modeOf_Payment_scroll">
          <view class="modeOf_Payment_box" v-for="(item,index) in 3" :key="index">
          <image src="../../static/images/224.jpg" />
          </view>
          </view>
          </scroll-view>
          </view>
          <view class="accomplish_contentbox_right">
          <view>共三件</view>
          <view class="accomplish_contentbox_right_money">￥<span>395.9</span></view>
          </view>
          </view>
          </view>
          
    view.one(v-show="active == 3")
      view.order(
        v-for="(item, index) in ReservedOrders",
        :key="index"
      )
        view.order-title
          view.order__date {{ item.date }}
          view.order__status {{ item.status }}
        view.order-content
          view.order__shop {{ item.title }}
          view.order__text {{ item.kidsCount }}儿童; {{ item.adultsCount }}成人

    // 详情  弹框
    uni-popup(ref="popup", type="center")
      view.gift_box
        view.gift_box_clear
          view.gift_box_clear_left 长宁天山店
          img.gift_box_clear_right(
            src="../../static/images/clear.png",
            @click="close()"
          )
        view.listdetail_box(v-for="(item, index) in 3", :key="index")
          view.listdetail_box_content
            img.listdetail_box_contentimg(
              src="../../static/images/224.jpg"
            )
            view.listdetail_box_content_titlebox
              view.listdetail_box_content_titlebox_left
                view.listdetail_box_content_titlebox_left_name 猪肋排
                view.listdetail_box_content_titlebox_left_number X 1
              view.listdetail_box_content_titlebox_right ¥ 140
        view.gift_box_clear.count_box
          view.gift_box_clear_left
          view.count_box_right 总计 ¥ 560
</template>

<script>
import uniPopup from '@/components/uni-popup/uni-popup.vue';
import tabs from '@/components/tabs/tabs.vue';
import cutomsTabs from '../../components/custom-tabs/tabs.vue';
export default {
  components: {
    tabs,
    uniPopup,
    'custom-tabs': cutomsTabs
  },
  data() {
    return {
      tabs: [
        {
          name: '已点餐'
        },
        {
          name: '已预约'
        },
        {
          name: '已完成'
        },
        {
          name: '已取消'
        }
      ],
      orderlists: [],
      active: 0,
      ReservedOrders: [], //已取消   已完成  已预约
      QRCodeHide: true //点餐二维码显示隐藏
    };
  },
  onLoad(option) {
    if (option.active) {
      this.active = 1;
    }
    this.getdetail('food');
  },
  methods: {
    // 切换tab
    changeTab(e) {
      if (e.index == 0) {
        console.log('已点餐');
        this.getdetail('food');
      } else if (e.index == 1) {
        this.getdetail('play', 'booked'); //已预约this
        console.log('已预约');
      } else if (e.index == 2) {
        this.getdetail('play', 'finishedd'); //已完成
        console.log('已完成');
      } else if (e.index == 3) {
        this.getdetail('play', 'canceled'); //已取消
        console.log('已取消');
      }
    },
    open() {
      this.$refs.popup.open();
    },
    close() {
      this.$refs.popup.close();
    },
    getdetail(type, status) {
      let info = {};
      if (status) {
        info = {
          type,
          status
        };
      } else {
        info = {
          type
        };
      }
      this.$axios.getRequest('/booking', info).then(res => {
        this.ReservedOrders = res; //获取订单列表
        //判断数组是否有长度
        if (this.ReservedOrders.length) {
          this.QRCodeHide = false;
          console.log('数组有长度，二维码隐藏');
        } else {
          this.QRCodeHide = true;
          console.log('数组为空，二维码显示');
        }

        // console.log(this.ReservedOrders,"this.ReservedOrders")
        res.forEach((r, index) => {
          if (r.status == 'pending') {
            r.status = '待付款';
          } else if (r.status == 'booked') {
            r.status = '已确认';
          } else if (r.status == 'in_service') {
            r.status = '已入场';
          } else if (r.status == 'pending_refund') {
            r.status = '待撤销';
          } else if (r.status == 'finished') {
            r.status = '已完成';
          } else if (r.status == 'canceled') {
            r.status = '已取消';
          }
        });
      });
    }
  }
};
</script>

<style lang="less" scoped>
.orderLlist {
  width: 750rpx;
  min-height: 100vh;
  background-color: #f8f8f8;

  // 已点餐
  .haveOrder_box {
    .haveOrder_scan_box {
      .haveOrder_scan {
        text-align: center;
        margin-top: 220rpx;

        .haveOrder_scan-code {
          width: 300rpx;
          height: 300rpx;
          margin: 0 auto;
          background: var(--theme--bg-main-color);
          border-radius: var(--theme--border-radius);
        }

        .haveOrder_scan_content {
          width: 168rpx;
          height: 68rpx;
          font-size: var(--theme--font-size-s);
          line-height: 42rpx;
          margin: 52rpx auto 0;
        }

        .haveOrder_scan_btn {
          width: 322rpx;
          height: 114rpx;
          background: var(--theme--main-color);
          border-radius: var(--theme--border-radius);
          margin: 0 auto;
          margin-top: 86rpx;
          font-size: var(--theme--font-size-m);
          line-height: 114rpx;
        }
      }
    }

    .accomplish_border {
      .accomplish_box {
        border: 1px solid red;
        margin-top: 20rpx;
        width: 710rpx;
        height: 320rpx;
        background: #ffffff;
        border-radius: 20rpx;

        .accomplish_top {
          margin-right: 20rpx;
          margin-left: 20rpx;
          padding-top: 25rpx;

          .accomplish_top_title_box {
            height: 100rpx;
            display: flex;
            justify-content: space-between;

            .accomplish_top_title {
              .accomplish_top_titlename {
                font-size: 30rpx;

                color: #222222;
              }

              span {
                font-size: 20rpx;

                color: #666666;
              }
            }

            .accomplish_top_btn_box {
              width: 128rpx;
              height: 36rpx;
              font-size: 26rpx;

              color: #666666;
              line-height: 125rpx;

              .accomplish_top_btn {
                width: 10rpx;
                height: 20rpx;
                margin-left: 20rpx;
              }
            }
          }
        }

        .accomplish_contentbox {
          margin-right: 20rpx;
          margin-left: 20rpx;
          margin-top: 25rpx;
          display: flex;
          justify-content: space-between;

          .accomplish_contentbox_left {
            width: 450rpx;

            .modeOf_Payment-box {
              .modeOf_Payment_scroll {
                display: flex;
                // width: 690rpx;
                margin: 0 auto;

                .modeOf_Payment_box {
                  margin: 0 10rpx 0 0;

                  image {
                    width: 120rpx;
                    height: 120rpx;
                    border-radius: 10rpx;
                  }
                }
              }
            }
          }

          .accomplish_contentbox_right {
            margin-top: 80rpx;
            width: 210rpx;
            display: flex;
            justify-content: space-between;

            view {
              width: 72rpx;
              height: 34rpx;
              font-size: 24rpx;

              color: #666666;
              line-height: 34rpx;
            }

            .accomplish_contentbox_right_money {
              width: 114rpx;
              height: 48rpx;
              font-size: 22rpx;

              color: #222222;
              line-height: 32rpx;

              span {
                font-size: 34rpx;
              }
            }
          }
        }
      }
    }
  }

  // 已预约
  .appointment_box {
    margin: 20rpx 0;
    width: 650rpx;
    height: 160rpx;
    background: #ffffff;
    border-radius: 20rpx;
    padding: 15rpx;

    .appointment_box_title {
      display: flex;
      justify-content: space-between;
      margin: 15rpx 5rpx 10rpx 5rpx;

      view {
        height: 44rpx;
        font-size: 32rpx;

        color: #666666;
        line-height: 44rpx;
      }
    }

    .appointment_box_name {
      height: 84rpx;
      font-size: 30rpx;

      color: #222222;
      line-height: 42rpx;

      .appointment_box_nameTitle {
        width: 470rpx;
        overflow: hidden;
        white-space: nowrap;
        text-overflow: ellipsis;
      }
    }
  }

  //已完成
  .accomplish_border {
    .accomplish_box {
      margin-top: 20rpx;
      width: 710rpx;
      height: 320rpx;
      background: #ffffff;
      border-radius: 20rpx;

      .accomplish_top {
        margin-right: 20rpx;
        margin-left: 20rpx;
        padding-top: 25rpx;

        .accomplish_top_title_box {
          height: 100rpx;
          display: flex;
          justify-content: space-between;

          .accomplish_top_title {
            .accomplish_top_titlename {
              font-size: 30rpx;

              color: #222222;
            }

            span {
              font-size: 20rpx;

              color: #666666;
            }
          }

          .accomplish_top_btn_box {
            width: 128rpx;
            height: 36rpx;
            font-size: 26rpx;

            color: #666666;
            line-height: 125rpx;

            .accomplish_top_btn {
              width: 10rpx;
              height: 20rpx;
              margin-left: 20rpx;
            }
          }
        }
      }

      .accomplish_contentbox {
        margin-right: 20rpx;
        margin-left: 20rpx;
        margin-top: 25rpx;
        display: flex;
        justify-content: space-between;

        .accomplish_contentbox_left {
          width: 450rpx;

          .modeOf_Payment-box {
            .modeOf_Payment_scroll {
              display: flex;
              // width: 690rpx;
              margin: 0 auto;

              .modeOf_Payment_box {
                margin: 0 10rpx 0 0;

                image {
                  width: 120rpx;
                  height: 120rpx;
                  border-radius: 10rpx;
                }
              }
            }
          }
        }

        .accomplish_contentbox_right {
          margin-top: 80rpx;
          width: 210rpx;
          display: flex;
          justify-content: space-between;

          view {
            width: 72rpx;
            height: 34rpx;
            font-size: 24rpx;

            color: #666666;
            line-height: 34rpx;
          }

          .accomplish_contentbox_right_money {
            width: 114rpx;
            height: 48rpx;
            font-size: 22rpx;

            color: #222222;
            line-height: 32rpx;

            span {
              font-size: 34rpx;
            }
          }
        }
      }
    }
  }
}

.gift_box {
  width: 602rpx;
  min-height: 808rpx;
  background: #ffffff;
  border-radius: 38rpx;

  .gift_box_clear {
    padding-top: 30rpx;
    width: 540rpx;
    margin: 35rpx auto;
    display: flex;
    justify-content: space-between;

    .gift_box_clear_left {
      width: 150rpx;
      height: 42rpx;
      font-size: 30rpx;

      color: #222222;
      line-height: 42rpx;
    }

    .gift_box_clear_right {
      width: 40rpx;
      height: 40rpx;
    }

    .count_box_right {
      width: 162rpx;
      height: 42rpx;
      font-size: 30rpx;

      color: #0d0d0d;
      line-height: 42rpx;
    }
  }

  .count_box {
    padding-bottom: 25rpx;
  }

  .listdetail_box {
    width: 540rpx;
    margin: 0 auto;

    .listdetail_box_content {
      margin-top: 40rpx;
      display: flex;
      justify-content: space-between;

      .listdetail_box_contentimg {
        width: 100rpx;
        height: 100rpx;
        border-radius: 20rpx;
      }

      .listdetail_box_content_titlebox {
        width: 410rpx;
        display: flex;
        justify-content: space-between;

        .listdetail_box_content_titlebox_left {
          .listdetail_box_content_titlebox_left_name {
            width: 90rpx;
            height: 42rpx;
            font-size: 30rpx;

            color: #0d0d0d;
            line-height: 42rpx;
          }

          .listdetail_box_content_titlebox_left_number {
            width: 40rpx;
            height: 42rpx;
            font-size: 26rpx;

            color: #999999;
            line-height: 75rpx;
          }
        }

        .listdetail_box_content_titlebox_right {
          height: 42rpx;
          font-size: 30rpx;

          color: #0d0d0d;
          line-height: 42rpx;
        }
      }
    }
  }
}

.order {
  width: 690rpx;
  border-radius: var(--theme--border-radius);
  box-shadow: var(--theme--box-shadow);
  margin: 30rpx auto 0;
  padding: 0 36rpx;
  box-sizing: border-box;
  background: white;
}

.order-title {
  display: flex;
  align-items: center;
  height: 90rpx;
  justify-content: space-between;
  border-bottom: 2rpx solid #d9dcdd;
  font-size: var(--theme--font-size-s);
}

.order__status {
  font-weight: var(--theme--font-weight-light);
}

.order-content::after {
  content: '';
  display: table-cell;
}

.order__shop {
  margin-top: 34rpx;
  font-size: var(--theme--font-size-m);
}

.order__list {
  display: flex;
  align-items: center;
  margin-top: 32rpx;
  margin-bottom: 34rpx;
}

.order__goods {
  display: flex;
  align-items: center;
  flex: 1;
}

.order__goods__item {
  width: 100rpx;
  height: 100rpx;
  border-radius: var(--theme--border-radius);
  background-color: var(--theme--bg-main-color);
}

.order__goods__item + .order__goods__item {
  margin-left: 16rpx;
}

.order__total {
  font-size: var(--theme--font-size-s);
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  align-items: flex-end;
  height: 100rpx;
}

.order__total__num {
  margin-bottom: 20rpx;
}

.order__text{
  margin-bottom: 40rpx;
  margin-top: 20rpx;
}
</style>
 