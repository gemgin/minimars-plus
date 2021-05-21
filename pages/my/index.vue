<template lang="pug">
view.my_box
  view.my_header-top(@click="goMarsCover()")
    img(:src="imgulr", mode="aspectFill")
  view.my_header_title_box
    view.my_header_titleBox
      view.my_header_title {{ nickname }}
    // MARS会员
    view.my_marsMember_box
      view.my_marsMemberheader
        view.my_marsMembertitle MARS会员
        view.my_marsMember_headerimg
          img(:src="heading", mode="aspectFill")
      view.my_marsMember_content-box
        view.my_marsMember-one
          img(src="../../static/images/my/my_jifen.png")
          view {{ points }}
          span MARS积分
        view.my_marsMember-one(@click="goCardBag")
          img(src="../../static/images/my/my_youhui.png")
          view {{ couPonNumber }}
          span MARS优惠券
        view.my_marsMember-one(@click="goCardBag")
          img(src="../../static/images/my/my_card.png")
          view {{ allCardBag }}
          span MARS卡包
  view.vip
  view.my_centre
    // 门店动态
    //- view.marsCard_contentbox-content
      view.my_centre_boxone 门店动态
      scroll-view.modeOf_Payment-box(scroll-x="true")
        view.modeOf_Payment_scroll
          view.modeOf_Payment_box(
            v-for="(item, index) in storeDynamic",
            :key="index",
            @click="gobuyCards"
          )
            // <view class="modeOf_Payment_box">
            view.my_centre_modeOf_PaymentBox
              img.modeOf_PaymentBox_leftimg(:src="item.posterUrl")
              view.modeOf_PaymentBox_right
                view.modeOf_PaymentBox_right_title {{ item.title }}
                view.modeOf_PaymentBox_right_content
                  rich-text(:nodes="item.content")
                  // 详情
                  // 任务中心
                  view.my_centre_boxtwo(@click="goTask")
                    view.my_centretwo
                      span 任务中心
  // 会员码
  view.my_vip_content
    view.my_vip_content_box
      view.my_vip_content_namebox(@click="goVipCode")
        view.menu__img.img-box
          img
        span 会员码
        img(src="../../static/images/111.png")
      view.my_vip_content_namebox(@click="goOrderList")
        view.menu__img.img-box
          img
        span 我的订单
        img(src="../../static/images/111.png")
      //- view.my_vip_content_namebox(@click="go")
        view.menu__img.img-box
          img
        span 消费历史
        img(src="../../static/images/111.png")
      //- view.my_vip_content_namebox
        view.menu__img.img-box
          img
        span 积分兑换
        img(src="../../static/images/111.png")
      //- view.my_vip_content_namebox
        view.menu__img.img-box
          img
        span 卡券兑换
        img(src="../../static/images/111.png")
      //- view.my_vip_content_namebox(@click="goCover")
        view.menu__img.img-box
          img
        span MARS封面
        img(src="../../static/images/111.png")
      view.my_vip_content_namebox(@click="goMore")
        view.menu__img.img-box
          img
        span 更多
        img(src="../../static/images/111.png")
</template>

<script>
export default {
  data() {
    return {
      phoneNumber: "", //手机号码
      heading: "",
      UserInformation: [], //用户信息
      showbanner: "",
      imgulr: "", //封面图
      nickname: "",
      defaulturl: "../../static/images/my/my_bj.png",
      couPonNumber: 0, //卡券数量
      allCardBag: 0, //卡包内数量
      points: 0, //积分
      storeDynamic: [], //门店动态
    };
  },
  onShow() {
    if (!this.nickname) {
      this.imgulr = this.defaulturl;
    }
    this.user();
    this.getCoupon();
    this.getCardBag();
    this.getArticleList();
  },
  methods: {
    // 获取文章列表
    getArticleList() {
      this.$axios
        .getRequest("/post", {
          tag: "store",
        })
        .then((res) => {
          console.log(res, "文章列表");
          this.storeDynamic = res;
        });
    },
    goMarsCover() {
      uni.navigateTo({
        url: "/pages/my/myCover?active=1",
      });
    },
    // 获取用户信息，接口
    user() {
      this.$axios.getRequest("/auth/user").then((res) => {
        this.UserInformation = res;
        this.points = res.points;
        let defaulturl = "../../static/images/my/my_bj.png";
        if (this.UserInformation.currentCover) {
          this.imgulr = this.UserInformation.currentCover.posterUrl;
        } else {
          this.imgulr = defaulturl;
        }
        // console.log(res.currentCover,"res.currentCover")
        if (res.currentCover) {
          this.showbanner = true;
        } else {
          this.showbanner = false;
        }
        // this.showbanner=res.currentCover
      });
    },
    // 获取手机号
    getphonenumber() {
      this.close();
    },
    // 优惠券
    getCoupon() {
      this.$axios
        .getRequest("/card", {
          type: "coupon",
        })
        .then((res) => {
          console.log(res.length, "优惠券");
          this.couPonNumber = res.length;
        });
    },
    // 卡包
    getCardBag() {
      this.$axios.getRequest("/card").then((res) => {
        console.log(res.length, "卡包");
        this.allCardBag = res.length;
      });
    },
    goCardBag() {
      uni.navigateTo({
        url: "/pages/my/cards",
      });
    },
    goOrderList() {
      uni.navigateTo({
        url: "/pages/my/bookings",
      });
    },
    goTask() {
      uni.navigateTo({
        url: "/pages/my/task",
      });
    },
    goVipCode() {
      uni.navigateTo({
        url: "/pages/my/code",
      });
    },
    goMore() {
      uni.navigateTo({
        url: "/pages/my/more",
      });
    },
    goCover() {
      uni.navigateTo({
        url: "/pages/my/cover",
      });
    },
  },
};
</script>

<style lang="less" scoped>
.my_box {
  background-color: #f8f8f8;

  .my_header-top {
    position: relative;
    width: 750rpx;
    height: 500rpx;

    image {
      width: 750rpx;
      height: 500rpx;
    }
  }

  .my_header_title_box {
    position: absolute;
    bottom: 140rpx;
    width: 750rpx;
    // height: 160rpx;
    position: relative;

    .my_header_titleBox {
      opacity: 0.7;
      // background: #D8D8D8;
      background-color: rgba(216, 216, 216, 0.6);
      width: 750rpx;
      height: 160rpx;

      .my_header_title {
        margin: 0 auto;
        width: 680rpx;
        height: 100rpx;
        font-size: var(--theme--font-size-l );
        color: #ffffff;
        line-height: 100rpx;
      }
    }

    // MARS会员
    .my_marsMember_box {
      // z-index: 5;
      position: absolute;
      margin-top: 0;
      right: 30rpx;
      top: 95rpx;
      width: 690rpx;
      height: 400rpx;
      background: #ffffff;
      box-shadow: 0px 4rpx 4rpx 4rpx rgba(0, 0, 0, 0.03);
      border-radius: 20rpx;

      .my_marsMemberheader {
        display: flex;
        width: 650rpx;
        justify-content: space-between;
        position: relative;

        .my_marsMembertitle {
          margin: 15rpx 0 0 30rpx;
          width: 178px;
          height: 50rpx;
          font-size: var(--theme--font-size-l);
          line-height: 50rpx;
        }

        .my_marsMember_headerimg {
          position: absolute;
          right: 20rpx;
          bottom: -10rpx;
          width: 140rpx;
          height: 140rpx;
          background: #bababa;
          border-radius: 140rpx;

          image {
            width: 140rpx;
            height: 140rpx;
            border-radius: 80rpx;
          }
        }
      }

      .my_marsMember_content-box {
        width: 650rpx;
        height: 400rpx;
        margin: 0 auto;
        display: flex;
        justify-content: space-around;

        .my_marsMember-one {
          width: 150rpx;
          height: 200rpx;
          margin-top: 70rpx;
          text-align: center;

          view {
            margin: 10rpx 0;
            font-size: var(--theme--font-size-m);
            line-height: 36rpx;
          }

          image {
            width: 100rpx;
            height: 100rpx;
            border-radius: 12rpx;
          }

          span {
            width: 145rpx;
            height: 34rpx;
            font-size: var(--theme--font-size-s);
            line-height: 34rpx;
          }
        }
      }
    }
  }

  .vip {
    height: 200rpx;
  }

  // 门店动态
  .my_centre {
    background: #f8f8f8;

    // 门店动态
    .marsCard_contentbox-content {
      width: 750rpx;
      margin: 20rpx 0 20px 27rpx;
      margin: 30px auto;

      // 标题
      .my_centre_boxone {
        margin: 15rpx auto;
        width: 600rpx;
        height: 42rpx;
        font-size: var(--theme--font-size-s);
        line-height: 42rpx;
      }

      .modeOf_Payment-box {
        .modeOf_Payment_scroll {
          display: flex;

          // justify-content: space-between;
          .modeOf_Payment_box {
            width: 690rpx;
            margin: 0 auto;
            height: 146rpx;
            background: #ffffff;
            box-shadow: 0rpx 4rpx 4rpx 4rpx rgba(0, 0, 0, 0.03);
            border-radius: 20rpx;
            margin: 0 25rpx;

            .my_centre_modeOf_PaymentBox {
              display: flex;
              align-items: center;
              justify-content: space-around;
              width: 600rpx;
              margin: 0 auto;

              .modeOf_PaymentBox_leftimg {
                width: 120rpx;
                height: 120rpx;
                border-radius: 120rpx;
                margin-top: 15rpx;
              }

              .modeOf_PaymentBox_right {
                width: 420rpx;

                .modeOf_PaymentBox_right_content {
                  height: 80rpx;
                  width: 420rpx;
                  overflow: hidden;
                  // white-space: nowrap;
                  text-overflow: ellipsis;
                }
              }
            }
          }
        }
      }
    }

    .my_centre_boxtwo {
      width: 690rpx;
      margin: 0 auto;
      margin-bottom: 30rpx;

      .my_centretwo {
        height: 146rpx;
        background: #ffffff;
        box-shadow: 0rpx 4rpx 4rpx 4rpx rgba(0, 0, 0, 0.03);
        border-radius: 20rpx;
        line-height: 146rpx;

        span {
          margin-left: 50rpx;
          line-height: 42rpx;
        }
      }
    }
  }

  // 会员码
  .my_vip_content {
    margin-top: 20rpx;
    width: 750rpx;
    height: 800rpx;
    background: #ffffff;

    .my_vip_content_box {
      width: 690rpx;
      margin: 0 auto;

      .my_vip_content_namebox {
        display: flex;
        justify-content: space-between;
        padding: 15px 20rpx;
        border-bottom: 1rpx solid #f3f3f3;
        align-items: center;

        span {
          width: 150rpx;
          height: 40rpx;
          font-size: var(--theme--font-size-s);
          line-height: 40rpx;
          flex-grow:1;
          margin-left: 20rpx;
        }

        image {
          width: 10rpx;
          height: 20rpx;
          margin-right: 30rpx;
          line-height: 20rpx;
        }
      }
    }
  }

  // 微信授权弹框
  .login_box {
    background-color: #ffffff;
    width: 750rpx;
    height: 550rpx;
    background: #ffffff;
    border-radius: 40rpx 40rpx 0rpx 0rpx;

    .login_box_clear {
      padding-top: 30rpx;
      width: 672rpx;
      margin: 0 auto;
      display: flex;
      justify-content: space-between;

      .login_box_clear_left {
        width: 40rpx;
        height: 40rpx;
      }

      .login_box_clear_right {
        width: 40rpx;
        height: 40rpx;
      }
    }
  }
}

.menu__img{
  width: 40rpx;
  height: 40rpx;
  background-color: var(--theme--bg-main-color);
}
</style>
