<script>
export default {
  onLaunch: function() {
    console.log('App Launch - uni-mall')
  },
  onShow: function() {
    console.log('App Show')
  },
  onHide: function() {
    console.log('App Hide')
  }
}
</script>

<style lang="scss">
/* 全局样式重置 */
page {
  font-family: -apple-system, BlinkMacSystemFont, 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', sans-serif;
  background-color: #F5F5F5;
  color: #333333;
  box-sizing: border-box;
}

/* 通用样式变量 */
/* 主色调：淘宝橙 #FF5000 */
/* 辅助橙：#FF8C00 */
/* 背景灰：#F5F5F5 */
/* 文字主色：#333333 */
/* 文字副色：#666666 */
/* 文字弱色：#999999 */

view, text, input, button, image {
  box-sizing: border-box;
}

/* 禁用点击高亮 */
button {
  background: none;
  border: none;
  padding: 0;
  margin: 0;
  line-height: 1;

  &::after {
    border: none;
  }
}

/* 防止文字选中 */
text {
  -webkit-user-select: none;
  user-select: none;
}
</style>