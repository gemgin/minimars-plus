<template lang="pug">
view.myOrder_box
  // 门店选择
  view.myOrder_top
    view.storesToChoose
      view.storesToChoose_choose
        view.title
          img(src="../../static/images/index/index_orderOne.png")
          view 门店选择
        view.content(@click="showShopPop")
          span {{store}} 
          img(src="../../static/images/search.png")
        view.title
          img(src="../../static/images/index/index_orderTwo.png")
          view 到访时间
        view.content(@click="showCalendarPop")
          span {{date[0]}}
          img.content_right(src="../../static/images/right.png")
        view.title
          img(src="../../static/images/index/index_orderThree.png")
          view 进场人数
        view.content(@click="showPeoplePop")
          span {{ adultsKidsText }}
          img.content_right(src="../../static/images/right.png")
  view.orderSpace
  view.modeOf_Payment
    view.modeOf_Payment_title
      view.img-box
        img(src="../../static/images/booking/more_payment.png")
      span 更多优惠支付方式
    view.modeOf_Payment_scroll
      view.modeOf_Payment_box(
        v-for="(item, index) in cards",
        :key="index",
        @click="selectCard(index)"
      )
        view.payment-box__img.img-box
          img(
            mode="aspectFill",
            :src="item.posterUrl",
            :class="[selectedCardIndex == index ? 'selected' : '', !isCardAvailable(item) ? 'disabled' : '']"
          )
        view.modeOf_Payment_box_name
          | {{ item.title }}
          view.payment-box__price
            | RMB {{ item.price }}
      view.modeOf_Payment_box.payment-box--add(@click="goBuyCards")
        view.payment-box__img.img-box
          view.payment-box__img--add.img-box
            img(src="../../static/images/booking/create_add.png")
        view.modeOf_Payment_box_name
          | 点击购买新卡
          
    // 订单支付
    view.pay-bar
      view.pay-bar__text 还需支付：{{ price }} 元
      // 
      view.pay-bar__btn(@click="pay")
          | 订单支付 PAYMENT
  
  // 门店选择
  custom-popup(ref="shopPop")
    view.pop-header(slot="header") 门店选择 STORES
    view(slot="body")
      custom-picker(valueKey="id",labelKey="name",:options="[stores]",@onchange="selectStore")
  
  // 日期选择
  custom-popup(ref="calendarPop")
    view.pop-header(slot="header")
      view.pop-header__arrow.pop-header__arrow--left.cover-mask--medium(@click="changeMonth(-1)")
      view {{currentMonth}}
      view.pop-header__arrow.pop-header__arrow--right.cover-mask--medium(@click="changeMonth(+1)")
    view.calendar__body(slot="body")
      custom-calendar(:markDays.sync="date",:displayMonth.sync="calendarDisplayMonth",ref="calendar")
  
  // 进场人数选择
  custom-popup(ref="peoplePop")
    view.pop-header(slot="header")
      view.people-pop__title 儿童 KIDS
      view.people-pop__title 成人 ADULTS
    view(slot="body")
      custom-picker(valueKey="value",labelKey="label",:options="adultsKidsValues",@onchange="selectAdultsKidsCount")
      
  // 礼品卡  弹框
  uni-popup(ref="cardContentPopup", type="center")
    view.gift_box
      view.gift_box_clear
        view.gift_box_clear_left
        img.gift_box_clear_right(
          src="../../static/images/clear.png",
          @click="closeCardContent()"
        )
      view.gift_contentBox
        view.gift_contentBox_title 礼品卡使用说明
        scroll-view.gift_contentBox_box(scroll-y="true")
          view.gift_contentBox_boxContent
            rich-text(:nodes="cardContent")
        view.gift_contentBox_btn(@click="closeCardContent")
          | 信息确认 SUBMIT
</template>

<script>
import { sync } from 'vuex-pathify';
import moment from 'moment';

import customPopup from '../../components/custom-popup/popup';
import customPicker from '../../components/custom-picker/picker';
import customCalendar from '../../components/custom-calendar/calendar';

export default {
  components: {
    'custom-popup': customPopup,
    'custom-picker': customPicker,
    'custom-calendar': customCalendar
  },
  data() {
    return {
      store: '',
      index: 0,
      stores: [],
      price: 0,
      adultsKidsValues: [[...Array(10).keys()].map(n => ({ label: n + 1, value: n + 1 })), [...Array(10).keys()].map(n => ({ label: n + 1, value: n + 1 }))],
      date: [moment().format('YYYY-MM-DD')],
      calendarDisplayMonth: moment().format('YYYY-MM-DD'),
      selectedCardIndex: -1,
      booking: {
        type: 'play',
        store: ' ', //门店ID
        date: '', //到访时间
        kidsCount: 1, //儿童人数
        adultsCount: 1, //成人人数
        card: '' //礼品卡ID
      },
      cards: [], //礼品卡
      cardContent: '', //礼品卡使用说明
      cardId: ''
    };
  },
  computed: {
    user: sync('auth/user'),
    adultsKidsText() {
      return this.booking.kidsCount + ' 儿童' + '，' + this.booking.adultsCount + ' 成人';
    },
    currentMonth() {
      return moment(this.calendarDisplayMonth).format('YYYY 年 MM 月');
    }
  },
  onLoad(option) {
    if (this.user.store) {
      this.store = this.user.store.name;
      this.booking.store = this.store.id;
    } else {
      this.store = '';
    }

    if (option.date) {
      this.date = [option.date];
    }

    this.booking.date = this.date[0];

    this.goStore();
    this.goCard();
    this.getPrice();
  },

  methods: {
    // 门店
    goStore() {
      this.$axios.getRequest('/store').then(res => {
        this.stores = res;
        this.store = res[0].name;
        this.booking.store = res[0].id;
      });
    },
    // 获取门店ID
    getStore() {
      this.stores.forEach(item => {
        if (item.name == this.store) {
          this.booking.store = item.id;
          this.getPrice();
        }
      });
    },
    // 卡片
    goCard() {
      this.$axios.getRequest('/card').then(res => {
        this.cards = res;
      });
    },
    // 去购卡
    goBuyCards() {
      this.goCard();
      uni.navigateTo({
        url: '../card/index'
      });
    },
    // 计算价格
    getPrice() {
      this.$axios.postRequest('/booking-price', this.booking).then(res => {
        this.price = res.price;
      });
    },

    // 订单支付
    async pay() {
      this.booking.date = this.date[0]; //时间
      this.booking.type = 'play';
      uni.showLoading();
      const booking = await this.$axios.postRequest('/booking', this.booking);
      uni.hideLoading();
      if (booking.payments[0].payArgs) {
        //唤起微信支付
        uni.requestPayment({
          provider: 'wxpay',
          timeStamp: booking.payments[0].payArgs.timeStamp,
          nonceStr: booking.payments[0].payArgs.nonceStr,
          package: booking.payments[0].payArgs.package,
          signType: 'MD5',
          paySign: booking.payments[0].payArgs.paySign,
          success: function(res) {
            console.log('success:' + JSON.stringify(res));
            uni.showToast({
              title: '预约成功',
              duration: 2000
            });
            uni.redirectTo({ url: '../my/bookings' });
          },
          fail: function(err) {
            console.log('fail:' + JSON.stringify(err));
          }
        });
      } else {
        this.goOrder(); //跳转订单
        uni.showToast({
          title: '预约成功',
          duration: 2000
        });
      }
    },
    // 跳转订单页面
    goOrder() {
      uni.redirectTo({
        url: '../my/bookings?active=1'
      });
    },

    // 显示日历弹窗
    showCalendarPop() {
      this.$refs.calendarPop.open();
    },

    // 选择月份
    changeMonth(n) {
      this.$refs.calendar.addMonth(n);
    },

    //日历
    goCalendar() {
      uni.navigateTo({
        url: './calendar'
      });
    },

    // 显示门店弹窗
    showShopPop() {
      this.$refs.shopPop.open();
    },

    selectStore(e) {
      this.store = e.value[0].name;
      this.getStore();
      this.goCard();
      this.getPrice();
    },

    // 显示人数弹窗
    showPeoplePop() {
      this.$refs.peoplePop.open();
    },

    selectAdultsKidsCount(e) {
      this.booking.kidsCount = +e.value[0].label;
      this.booking.adultsCount = +e.value[1].label;
      this.getPrice();
    },
    selectCard(index) {
      const card = this.cards[index];
      if (!this.isCardAvailable(card)) return;

      if (this.selectedCardIndex === index) {
        this.selectedCardIndex = -1;
        this.booking.card = undefined;
      } else {
        this.selectedCardIndex = index;
        this.booking.card = card.id;
        this.showCardContent(card);
      }
      this.getPrice();
    },
    isCardAvailable(card) {
      if (!this.booking.store) return false;
      if (card.stores.length === 0) return true;
      if (card.stores.includes(this.booking.store)) return true;
      return false;
    },
    showCardContent(card) {
      this.cardContent = card.content;
      this.$refs.cardContentPopup.open();
    },
    closeCardContent() {
      this.$refs.cardContentPopup.close();
    }
  }
};
</script>

<style lang="less" scoped>
page {
  background-color: #f8f8f8;
}

.myOrder_box {

  // 门店选择
  .myOrder_top {
    position: relative;
    width: 750rpx;
    height: 330rpx;
    background: #d8d8d8;

    .storesToChoose {
      height: 520rpx;
      position: absolute;
      margin-top: 85rpx;
      right: 30rpx;
      width: 690rpx;
      display: flex;
      justify-content: space-around;
      background: #ffffff;
      box-shadow: 0rpx 4rpx 4rpx 4rpx rgba(0, 0, 0, 0.03);
      border-radius: var(--theme--border-radius);

      .storesToChoose_choose {
        height: 170rpx;

        .title {
          display: flex;
          margin-top: 20rpx;

          image {
            width: 40rpx;
            height: 40rpx;
          }

          view {
            margin-left: 16rpx;
            width: 112rpx;
            height: 40rpx;
            font-size: var(--theme--font-size-m);
            color: var(--theme--font-deputy-color);
            line-height: 40rpx;
          }
        }

        .content {
          width: 594rpx;
          height: 80rpx;
          line-height: 80rpx;
          border-radius: var(--theme--border-radius);
          border: 2rpx solid #aaaeaf;
          display: flex;
          justify-content: space-between;
          margin-top: 15rpx;
          font-size: var(--theme--font-size-m);
          color: var(--theme--font-main-color);

          input {
            margin-left: 30rpx;
            margin-top: 20rpx;
          }

          span {
            width: 550rpx;
            margin-left: 30rpx;
          }

          image {
            width: 31rpx;
            height: 31rpx;
            margin-right: 25rpx;
            margin-top: 28rpx;
          }
        }
      }
    }
  }

  .orderSpace {
    width: 750rpx;
    height: 300rpx;
  }

  // 更多优惠
  .modeOf_Payment {
    .modeOf_Payment_title {
      width: 690rpx;
      margin: 0 auto;
      font-size: var(--theme--font-size-m);
      color: var(--theme--font-deputy-color);
      margin-bottom: 20rpx;
      display: flex;
      align-items: center;

      .img-box {
        width: 32rpx;
        height: 26rpx;
        margin-left: 50rpx;
        margin-right: 20rpx;
      }
    }

    .modeOf_Payment_scroll {
      overflow-x: scroll;
      white-space: nowrap;

      .modeOf_Payment_box {
        display: inline-block;
        vertical-align: top;
        margin-right: 38rpx;
        opacity: 1;

        &:first-child {
          margin-left: 38rpx;
        }

        .payment-box__img {
          width: 380rpx;
          height: 230rpx;
          border-radius: var(--theme--border-radius);
          position: relative;
          background-color: var(--theme--bg-main-color);
        }

        .payment-box__img--add {
          width: 68rpx;
          height: 68rpx;
          position: absolute;
          left: 50%;
          top: 50%;
          transform: translate(-50%, -50%);
        }

        .selected {
          border-bottom: 10rpx solid #42ff62;
          box-sizing: border-box;
        }

        .disabled {
          opacity: 0.5;
        }

        .modeOf_Payment_box_btn {
          width: 280rpx;
          height: 230rpx;
          border-radius: var(--theme--border-radius);
        }

        .modeOf_Payment_box_name {
          font-size: var(--theme--font-size-s);
          color: var(--theme--font-main-color);
          margin: 10rpx 0;

          .payment-box__price {
            font-size: var(--theme--font-size-m);
            line-height: 42rpx;
          }
        }
      }
    }

    .pay-bar {
      display: flex;
      justify-content: space-between;
      align-items: center;
      height: 120rpx;
      width: 100%;
      padding-bottom: env(safe-area-inset-bottom);
      position: fixed;
      bottom: 0;
      background-color: #f8f8f8;
      font-size: var(--theme--font-size-m);

      .pay-bar__text {
        color: var(--theme--font-main-color);
        margin-left: 40rpx;
      }

      .pay-bar__btn {
        background-color: var(--theme--main-color);
        height: 102rpx;
        line-height: 102rpx;
        padding: 0 46rpx;
        margin-right: 30rpx;
        border-radius: var(--theme--border-radius);
      }
    }
  }
}

.gift_box {
  width: 600rpx;
  height: 940rpx;
  background: #ffffff;
  border-radius: var(--theme--border-radius);

  .gift_box_clear {
    padding-top: 30rpx;
    width: 540rpx;
    margin: 0 auto;
    display: flex;
    justify-content: space-between;

    .gift_box_clear_left {
      width: 40rpx;
      height: 40rpx;
    }

    .gift_box_clear_right {
      width: 40rpx;
      height: 40rpx;
    }
  }

  .gift_contentBox {
    width: 600rpx;
    margin: 0 auto;
    text-align: center;

    .gift_contentBox_title {
      width: 220rpx;
      line-height: 42rpx;
      margin: 0 auto 20rpx;
    }

    .gift_contentBox_box {
      width: 570rpx;
      height: 690rpx;
      margin: 0 auto;

      .gift_contentBox_boxContent {
        text-align: left;
      }
    }

    .gift_contentBox_btn {
      height: 102rpx;
      line-height: 102rpx;
      padding: 0 62rpx;
      background: var(--theme--main-color);
      border-radius: var(--theme--border-radius);
      display: inline-block;
      white-space: nowrap;
    }
  }
}
</style>

<style scoped>
.pop-header,
.pop-header,
.pop-header {
  font-size: var(--theme--font-size-normal);
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
}

.people-pop__title {
  flex: 1;
  text-align: center;
}

.people-pop__title + .people-pop__title {
  border-left: 2rpx solid #6f8f7d;
}

.calendar__body {
  padding: 0 80rpx 25rpx;
  box-sizing: border-box;
}

.pop-header__arrow {
  width: 20rpx;
  height: 20rpx;
  background: url('../../static/images/booking/calendar_arrow.png') no-repeat center;
  background-size: contain;
}

.pop-header__arrow--left {
  transform: rotate(-90deg);
  margin-right: 40rpx;
}

.pop-header__arrow--right {
  transform: rotate(90deg);
  margin-left: 40rpx;
}
</style>
