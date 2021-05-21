<template lang="pug">
view
</template>

<script>
export default {
  data() {
    return {};
  },
  onShow() {
    uni.getUserInfo({
      provider: "weixin",
      success: (infoRes) => {
        console.log(infoRes, "用户信息");
      },
    });
  },
  methods: {},
};
</script>

<style></style>
