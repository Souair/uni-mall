<template>
  <view class="page-container">

    <view :style="{ height: statusBarHeight + 'px', background: '#FFb4c0' }"></view>
    <view class="layer-1-ad" v-if="showTopAd">
      <view class="ad-content">
        <view class="ad-nav-list">
          <view class="ad-nav-item" v-for="item in navItems" :key="item.id" @tap="onNavTap(item)">
            <view class="nav-icon">{{ item.icon }}</view>
            <text class="nav-text">{{ item.name }}</text>
          </view>
        </view>
      </view>
      <view class="close-btn" @tap="closeTopAd">✕</view>
    </view>

    <view class="layer-2-header">
      <view class="header-left-logo">
        <image class="logo-image-fill" src="/images/taobao.png" mode="heightFix"></image>
      </view>
      <view class="header-right-search">
        <view class="search-box">
          <view class="search-type-wrap" @tap="toggleSearchType">
            <text class="search-type-text">{{ searchType }}</text>
            <text class="search-type-arrow">▾</text>
          </view>
          <input
            class="search-input"
            v-model="searchKeyword"
            :placeholder="currentPlaceholder"
            placeholder-class="search-placeholder"
            @confirm="onSearch"
          />
          <view class="search-btn" @tap="onSearch">
            <text class="search-btn-text">搜索</text>
          </view>
        </view>
        <view class="hot-keywords">
          <text class="hot-kw-item" v-for="(kw, index) in hotKeywords" :key="index" @tap="onHotKwTap(kw)">{{ kw }}</text>
        </view>
      </view>
    </view>

    <view class="layer-3-main">

      <!-- 左侧分类 -->
      <view class="col-left-category">
        <view class="cat-title">分类</view>
        <view class="cat-list">
          <view
            v-for="(cat, index) in categories"
            :key="index"
            class="cat-item"
            :class="{ active: activeCat === index }"
            @tap="onCatTap(index)"
          >
            <text v-if="index === 0" class="cat-text highlight-cat">券后7.7折起 / 抢省钱补贴</text>
            <text v-else class="cat-text">{{ cat }}</text>
          </view>
        </view>
      </view>

      <!-- 中间轮播 -->
      <view class="col-center-banner" @mouseenter="isHover = true" @mouseleave="isHover = false">
        <swiper
          class="main-swiper"
          :indicator-dots="false"
          :autoplay="!isHover"
          :interval="3000"
          :circular="true"
          :current="swiperCurrent"
          @change="onSwiperChange"
        >
          <swiper-item v-for="banner in banners" :key="banner.id">
            <view class="banner-item" :style="{ background: banner.bg }" @tap="onBannerTap(banner)">
              <view class="banner-content">
                <text class="new-tag">新热周</text>
                <text class="banner-title">{{ banner.title }}</text>
                <text class="banner-subtitle">{{ banner.subtitle }}</text>
                <text class="banner-date">{{ banner.date }}</text>
              </view>
              <view class="banner-images">
                <view v-for="(img, i) in banner.mockImages" :key="i" class="banner-mock-img" :class="'img-pos-' + i">
                  <text class="banner-mock-icon">{{ img.icon }}</text>
                </view>
              </view>
            </view>
          </swiper-item>
        </swiper>

        <!-- 左右切换按钮 -->
        <view class="swiper-btn prev" v-show="isHover" @tap="prevSlide">〈</view>
        <view class="swiper-btn next" v-show="isHover" @tap="nextSlide">〉</view>

        <!-- 自定义圆点指示器（5个圆点） -->
        <view class="custom-dots">
          <view
            v-for="(banner, index) in banners"
            :key="index"
            class="dot"
            :class="{ 'dot-active': swiperCurrent === index }"
            @tap="goToSlide(index)"
          ></view>
        </view>
      </view>

      <!-- 右侧快捷入口 -->
      <view class="col-right-entries">
        <view
          v-for="entry in quickEntries"
          :key="entry.id"
          class="entry-card"
          :style="{ background: entry.color }"
          @tap="onEntryTap(entry)"
        >
          <view class="entry-text">
            <text class="entry-title">{{ entry.title }}</text>
            <text class="entry-sub">{{ entry.subtitle }}</text>
          </view>
          <view class="entry-icon-wrap">
            <text class="entry-icon">{{ entry.icon }}</text>
          </view>
        </view>
      </view>

    </view>

  </view>
</template>

<script>
export default {
  name: 'IndexPage',
  data() {
    return {
      isHover: false,
      swiperCurrent: 0,
      statusBarHeight: 0,
      showTopAd: true,
      searchKeyword: '',
      searchType: '宝贝',
      activeCat: 0,
      currentPlaceholder: '第一人称拍摄设备',

      navItems: [
        { id: 1, name: '国家补贴', icon: '🔰' },
        { id: 2, name: '淘宝秒杀', icon: '⏱' },
        { id: 3, name: 'U先试用', icon: 'U' },
        { id: 4, name: '百亿补贴来啦', icon: '💰' },
        { id: 5, name: '领券中心', icon: '券' },
        { id: 6, name: '百亿补贴', icon: '补' },
        { id: 7, name: '聚划算', icon: '聚' }
      ],

      hotKeywords: [
        '循环唱机', '安平丝网', '不锈钢筛网', '手机支架胸挂',
        '老人随身音响', '骑行手机支架', '钓鱼手机支架', '304不锈钢编织网', '防虫网'
      ],

      categories: [
        '券后7.7折起 / 抢省钱补贴',
        '电脑 / 配件 / 办公 / 文具',
        '家电 / 手机 / 通信 / 数码',
        '工业品 / 商业 / 农业 / 定制',
        '家具 / 家装 / 家居 / 厨具',
        '女装 / 男装 / 内衣 / 配饰',
        '女鞋 / 男鞋 / 运动 / 户外',
        '汽车 / 珠宝 / 文玩 / 箱包',
        '食品 / 鲜花 / 酒水 / 健康',
        '母婴 / 童装 / 玩具 / 宠物',
        '美妆 / 个护 / 娱乐 / 图书',
      ],

      // ===== 五张轮播图 =====
      banners: [
        {
          id: 1,
          title: '运动户外',
          subtitle: '春野爆款8.8折起',
          date: '03.23-03.31火爆热卖',
          bg: '#C58925',
          mockImages: [
            { icon: '🛹' },
            { icon: '🏈' },
            { icon: '📷' },
          ]
        },
        {
          id: 2,
          title: '数码好物',
          subtitle: '新品直降5折起',
          date: '03.23-03.31限时特惠',
          bg: 'linear-gradient(135deg, #1a237e, #3949AB)',
          mockImages: [
            { icon: '💻' },
            { icon: '📱' },
            { icon: '🎧' },
          ]
        },
        {
          id: 3,
          title: '家居生活',
          subtitle: '精选爆款7折起',
          date: '03.23-03.31超值抢购',
          bg: 'linear-gradient(135deg, #2E7D32, #66BB6A)',
          mockImages: [
            { icon: '🪴' },
            { icon: '🛋️' },
            { icon: '💡' },
          ]
        },
        {
          id: 4,
          title: '美妆护肤',
          subtitle: '大牌低至3折起',
          date: '03.23-03.31品质特惠',
          bg: 'linear-gradient(135deg, #880E4F, #E91E63)',
          mockImages: [
            { icon: '💄' },
            { icon: '🧴' },
            { icon: '✨' },
          ]
        },
        {
          id: 5,
          title: '食品生鲜',
          subtitle: '新鲜直达9折起',
          date: '03.23-03.31限时秒杀',
          bg: 'linear-gradient(135deg, #BF360C, #FF7043)',
          mockImages: [
            { icon: '🍎' },
            { icon: '🥩' },
            { icon: '🍜' },
          ]
        },
      ],

      quickEntries: [
        { id: 1, title: '品质家居', subtitle: '超值优惠', icon: '🛋️', color: '#FF5722' },
        { id: 2, title: '精致美妆', subtitle: '品质之选', icon: '💄', color: '#E91E63' },
        { id: 3, title: '品质五金', subtitle: '超值特惠', icon: '🧰', color: '#FF9800' },
        { id: 4, title: '超值百货', subtitle: '省钱省心', icon: '🧴', color: '#03A9F4' },
      ]
    }
  },

  onLoad() {
    // #ifdef APP-PLUS
    const info = uni.getSystemInfoSync()
    this.statusBarHeight = info.statusBarHeight
    // #endif
  },

  methods: {
    closeTopAd() { this.showTopAd = false },
    onNavTap(item) { uni.showToast({ title: item.name, icon: 'none' }) },
    toggleSearchType() { this.searchType = this.searchType === '宝贝' ? '店铺' : '宝贝' },
    onSearch() {
      if (!this.searchKeyword.trim()) { uni.showToast({ title: '请输入搜索关键词', icon: 'none' }); return }
      uni.navigateTo({ url: `/pages/search/search?keyword=${encodeURIComponent(this.searchKeyword)}` })
    },
    onHotKwTap(kw) { this.searchKeyword = kw; this.onSearch() },
    onCatTap(index) { this.activeCat = index },
    onBannerTap(banner) { uni.showToast({ title: banner.title, icon: 'none' }) },
    onEntryTap(entry) { uni.showToast({ title: entry.title, icon: 'none' }) },

    // ===== 轮播控制 =====
    onSwiperChange(e) {
      // UniApp swiper change事件同步当前索引
      this.swiperCurrent = e.detail.current
    },
    prevSlide() {
      this.swiperCurrent = (this.swiperCurrent - 1 + this.banners.length) % this.banners.length
    },
    nextSlide() {
      this.swiperCurrent = (this.swiperCurrent + 1) % this.banners.length
    },
    goToSlide(index) {
      this.swiperCurrent = index
    },
  }
}
</script>

<style lang="scss">
.page-container {
  min-height: 100vh;
  background: #ffffff;
  display: flex;
  flex-direction: column;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
}

/* ==================== 第一层：顶部广告 ==================== */
.layer-1-ad {
  position: relative;
  width: 100%;
  height: 120rpx;
  background: linear-gradient(90deg, #ff9ea7, #ff5c77);
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;

  .ad-content {
    display: flex;
    align-items: center;
    width: 100%;
    padding: 0 80rpx;
    justify-content: center;
  }

  .ad-nav-list {
    display: flex;
    align-items: center;
    gap: 40rpx;

    .ad-nav-item {
      display: flex;
      flex-direction: column;
      align-items: center;
      color: #fff;
      cursor: pointer;

      .nav-icon {
        font-size: 40rpx;
        background: #fff;
        border-radius: 50%;
        width: 50rpx;
        height: 50rpx;
        display: flex;
        align-items: center;
        justify-content: center;
        margin-bottom: 6rpx;
        color: #ff5c77;
      }
      .nav-text { font-size: 22rpx; }
    }
  }

  .close-btn {
    position: absolute;
    right: 30rpx;
    top: 50%;
    transform: translateY(-50%);
    width: 40rpx;
    height: 40rpx;
    border: 2rpx solid rgba(255,255,255,0.6);
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    color: #fff;
    font-size: 20rpx;
    cursor: pointer;
  }
}

/* ==================== 第二层：搜索 ==================== */
.layer-2-header {
  display: flex;
  align-items: flex-start;
  padding: 40rpx;
  background: #ffffff;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
  box-sizing: border-box;

  .header-left-logo {
    display: flex;
    align-items: center;
    margin-right: 60rpx;
    height: 80rpx;
    cursor: pointer;

    .logo-image-fill { height: 100%; width: auto; display: block; }
  }

  .header-right-search {
    flex: 1;
    display: flex;
    flex-direction: column;
    padding-top: 10rpx;

    .search-box {
      display: flex;
      align-items: center;
      border: 4rpx solid #FF5000;
      border-radius: 40rpx;
      height: 72rpx;
      overflow: hidden;
      width: 100%;
      max-width: 800px;

      .search-type-wrap {
        display: flex;
        align-items: center;
        padding: 0 24rpx;
        background: #f5f5f5;
        height: 100%;
        border-right: 2rpx solid #eee;
        cursor: pointer;
        .search-type-text { font-size: 26rpx; color: #333; }
        .search-type-arrow { font-size: 20rpx; color: #999; margin-left: 8rpx; }
      }

      .search-input { flex: 1; height: 100%; padding: 0 20rpx; font-size: 26rpx; }
      .search-placeholder { color: #999; }

      .search-btn {
        background: #FF5000;
        height: 100%;
        padding: 0 50rpx;
        display: flex;
        align-items: center;
        justify-content: center;
        cursor: pointer;
        .search-btn-text { color: #fff; font-size: 32rpx; font-weight: bold; letter-spacing: 4rpx; }
      }
    }

    .hot-keywords {
      display: flex;
      flex-wrap: wrap;
      gap: 16rpx;
      margin-top: 12rpx;
      .hot-kw-item { font-size: 22rpx; color: #999; cursor: pointer; &:hover { color: #FF5000; } }
    }
  }
}

/* ==================== 第三层：三栏布局 ==================== */
.layer-3-main {
  display: flex;
  align-items: stretch;
  padding: 0 40rpx 40rpx;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
  box-sizing: border-box;
  gap: 20rpx;
  height: 580rpx;

  /* 左侧分类 */
  .col-left-category {
    width: 320rpx;
    background: #f7f9fa;
    border-radius: 12rpx;
    padding: 20rpx 0;
    flex-shrink: 0;
    overflow-y: hidden;

    .cat-title { font-size: 28rpx; font-weight: bold; color: #333; padding: 0 20rpx 20rpx; }

    .cat-list {
      display: flex;
      flex-direction: column;

      .cat-item {
        padding: 12rpx 20rpx;
        cursor: pointer;
        &:hover, &.active { background: #ffebe5; .cat-text { color: #FF5000; } }

        .cat-text {
          font-size: 24rpx; color: #666;
          white-space: nowrap; overflow: hidden; text-overflow: ellipsis; display: block;
        }
        .highlight-cat { color: #FF5000; font-weight: bold; }
      }
    }
  }

  /* 中间轮播 */
  .col-center-banner {
    flex: 1;
    min-width: 0;
    border-radius: 16rpx;
    overflow: hidden;
    height: 100%;
    position: relative; /* 切换按钮和圆点定位的基准 */

    .main-swiper {
      width: 100%;
      height: 100%;

      .banner-item {
        width: 100%;
        height: 100%;
        padding: 40rpx;
        box-sizing: border-box;
        display: flex;
        justify-content: space-between;
        align-items: center;
        overflow: hidden;

        .banner-content {
          display: flex;
          flex-direction: column;
          z-index: 2;
          flex-shrink: 0;

          .new-tag {
            background: #FF5000; color: #fff; font-size: 20rpx;
            padding: 4rpx 12rpx; border-radius: 6rpx;
            align-self: flex-start; margin-bottom: 20rpx;
          }
          .banner-title { font-size: 50rpx; font-weight: bold; color: #fff; margin-bottom: 10rpx; }
          .banner-subtitle { font-size: 34rpx; color: #fff; margin-bottom: 20rpx; }
          .banner-date { font-size: 22rpx; color: rgba(255,255,255,0.8); }
        }

        /* 三图标叠放区域 */
        .banner-images {
          position: relative;
          width: 300rpx;
          height: 100%;
          flex-shrink: 0;

          .banner-mock-img {
            position: absolute;
            display: flex;
            align-items: center;
            justify-content: center;

            /* 左下角小图标 */
            &.img-pos-0 {
              left: 0;
              bottom: 20rpx;
              .banner-mock-icon { font-size: 80rpx; display: block; transform: rotate(-15deg); }
            }
            /* 中间大图标（最突出） */
            &.img-pos-1 {
              left: 80rpx;
              top: 50%;
              transform: translateY(-50%);
              .banner-mock-icon { font-size: 130rpx; display: block; }
            }
            /* 右下角小图标 */
            &.img-pos-2 {
              right: 0;
              bottom: 30rpx;
              .banner-mock-icon { font-size: 80rpx; display: block; transform: rotate(10deg); }
            }
          }
        }
      }
    }

    /* 左右切换按钮 */
    .swiper-btn {
      position: absolute;
      top: 50%;
      transform: translateY(-50%);
      width: 60rpx;
      height: 60rpx;
      background: rgba(0,0,0,0.35);
      color: #fff;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 28rpx;
      cursor: pointer;
      z-index: 20;
      transition: background 0.2s;
      user-select: none;
      &:hover { background: rgba(0,0,0,0.6); }
      &.prev { left: 16rpx; }
      &.next { right: 16rpx; }
    }

    /* 圆点指示器 */
    .custom-dots {
      position: absolute;
      bottom: 16rpx;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      gap: 10rpx;
      z-index: 20;

      .dot {
        width: 12rpx;
        height: 12rpx;
        border-radius: 50%;
        background: rgba(255,255,255,0.5);
        cursor: pointer;
        transition: all 0.3s ease;

        /* 激活状态：拉宽变白，药丸形 */
        &.dot-active {
          width: 30rpx;
          border-radius: 6rpx;
          background: #ffffff;
        }
      }
    }
  }

  /* 右侧快捷入口 */
  .col-right-entries {
    width: 260rpx;
    height: 100%;
    flex-shrink: 0;
    display: flex;
    flex-direction: column;
    gap: 16rpx;

    .entry-card {
      flex: 1;
      border-radius: 12rpx;
      padding: 0 24rpx;
      display: flex;
      justify-content: space-between;
      align-items: center;
      cursor: pointer;
      color: #fff;
      transition: transform 0.2s;
      &:hover { transform: translateY(-2rpx); box-shadow: 0 6rpx 16rpx rgba(0,0,0,0.15); }

      .entry-text {
        display: flex;
        flex-direction: column;
        .entry-title { font-size: 30rpx; font-weight: bold; margin-bottom: 4rpx; }
        .entry-sub { font-size: 20rpx; opacity: 0.9; }
      }

      .entry-icon-wrap {
        background: rgba(255,255,255,0.2);
        width: 70rpx;
        height: 70rpx;
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
        .entry-icon { font-size: 34rpx; }
      }
    }
  }
}
</style>