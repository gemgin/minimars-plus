<script>
import { sync } from 'vuex-pathify';
import wechatLogin from '@/utils/wechatLogin';

export default {
  computed: {
    auth: sync('auth')
  },
  async onLaunch() {
    console.log('App Launch');
    //全局配置
    this.config = await this.$axios.getRequest('/config');

    try {
      uni.showLoading({ mask: true });
      if (!this.auth.token) {
        const auth = await wechatLogin();
        Object.assign(this.auth, auth);
      } else {
        const user = await this.getAuthUser();
        this.auth.user = user;
      }
      // this.$launched();
      uni.hideLoading();
    } catch (e) {
      console.error(e);
    }
  },
  onShow() {
    console.log('App Show');
  },
  onHide() {
    console.log('App Hide');
  },
  methods: {
    async getAuthUser() {
      const user = await this.$axios.getRequest('/auth/user');
      console.log('Get auth user:', user);
      return user;
    }
  }
};
</script>

<style>
@import url('./utils/all.min.css');
page {
  --theme--main-color: #92ffac;
  --theme--deputy-color: #b9dcfc;
  --theme--font-size-s: 22rpx;
  --theme--font-size-m: 28rpx;
  --theme--font-size-l: 36rpx;
  --theme--font-main-color: #646a6d;
  --theme--font-deputy-color: #aaaeaf;
  --theme--bg-main-color: #d9dcdd;
  --theme--font-weight-light: 300;
  --theme--font-weight-medium: 500;
  --theme--border-radius: 20rpx;
  --theme--box-shadow: 3rpx 3rpx 8rpx 3rpx rgba(0, 0, 0, 0.1);
}

* {
  margin: 0;
  padding: 0;
}

html {
  font-weight: var(--theme--font-weight-medium);
  font-size: var(--theme--font-size-m);
  color: var(--theme--font-main-color);
  font-family: Gotham, 'PingFang SC', sans-serif;
}

@font-face {
  font-family: 'Gotham';
  src: url('~@/static/fonts/Gotham-Light.woff2') format('woff2'), url('~@/static/fonts/Gotham-Light.woff') format('woff');
  font-weight: 300;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: 'Gotham';
  src: url('~@/static/fonts/Gotham-Medium.woff2') format('woff2'), url('~@/static/fonts/Gotham-Medium.woff') format('woff');
  font-weight: 500;
  font-style: normal;
  font-display: swap;
}
</style>
