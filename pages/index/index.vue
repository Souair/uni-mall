<template>
  <view class="page-container">

    <!-- 状态栏占位，仅App端 -->
    <!-- #ifdef APP-PLUS -->
    <view :style="{ height: statusBarHeight + 'px' }" class="status-bar-placeholder"></view>
    <!-- #endif -->

    <!-- ===== 第一层：顶部导航广告条 ===== -->
    <view class="layer-1-ad" v-if="showTopAd">
      <scroll-view scroll-x class="ad-scroll" :show-scrollbar="false">
        <view class="ad-nav-list">
          <view class="ad-nav-item" v-for="item in navItems" :key="item.id" @tap="onNavTap(item)">
            <view class="nav-icon-wrap" :style="{ background: item.bg }">
              <text class="nav-icon">{{ item.icon }}</text>
            </view>
            <text class="nav-text">{{ item.name }}</text>
          </view>
          <!-- 百亿补贴横幅按钮 -->
          <view class="nav-banner-btn" @tap="onNavTap({ name: '百亿补贴' })">
            <text class="nav-banner-text">百亿补贴来啦</text>
          </view>
        </view>
      </scroll-view>
      <view class="close-btn" @tap="closeTopAd">✕</view>
    </view>

    <!-- ===== 第二层：Logo + 搜索框 ===== -->
    <view class="layer-2-header">
      <!-- PC端wrapper居中 -->
      <view class="header-inner">
        <view class="header-left-logo">
          <text class="logo-text">淘宝</text>
          <text class="logo-sub">Taobao.com</text>
        </view>
        <view class="header-right-search">
          <view class="search-box">
            <view class="search-type-wrap" @tap="toggleSearchType">
              <text class="search-type-text">{{ searchType }}</text>
              <text class="search-type-arrow">▾</text>
            </view>
            <view class="search-divider"></view>
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
          <!-- 热搜词 -->
          <view class="hot-keywords">
            <text
              class="hot-kw-item"
              v-for="(kw, index) in hotKeywords"
              :key="index"
              @tap="onHotKwTap(kw)"
            >{{ kw }}</text>
          </view>
        </view>
      </view>
    </view>

    <!-- ===== 手机端：Banner + 快捷入口 + 分类（纵向布局） ===== -->
    <!-- #ifndef H5 -->
    <view class="mobile-body">

      <!-- 手机端Banner轮播 -->
      <view class="mobile-banner-wrap" @touchstart="onTouchStart" @touchend="onTouchEnd">
        <swiper
          class="mobile-swiper"
          :indicator-dots="false"
          :autoplay="true"
          :interval="3000"
          :circular="true"
          :current="swiperCurrent"
          @change="onSwiperChange"
        >
          <swiper-item v-for="banner in banners" :key="banner.id">
            <view class="mobile-banner-item" :style="{ background: banner.bg }" @tap="onBannerTap(banner)">
              <view class="mobile-banner-content">
                <text class="mobile-new-tag">新热周</text>
                <text class="mobile-banner-title">{{ banner.title }}</text>
                <text class="mobile-banner-subtitle">{{ banner.subtitle }}</text>
                <text class="mobile-banner-date">{{ banner.date }}</text>
              </view>
              <view class="mobile-banner-icons">
                <view v-for="(img, i) in banner.mockImages" :key="i" class="mobile-banner-icon" :class="'mbi-' + i">
                  <text>{{ img.icon }}</text>
                </view>
              </view>
            </view>
          </swiper-item>
        </swiper>
        <!-- 圆点指示器 -->
        <view class="mobile-dots">
          <view
            v-for="(b, i) in banners"
            :key="i"
            class="mobile-dot"
            :class="{ 'mobile-dot-active': swiperCurrent === i }"
            @tap="goToSlide(i)"
          ></view>
        </view>
      </view>

      <!-- 手机端：快捷入口 2x2 -->
      <view class="mobile-entries">
        <view
          v-for="entry in quickEntries"
          :key="entry.id"
          class="mobile-entry-card"
          :style="{ background: entry.color }"
          @tap="onEntryTap(entry)"
        >
          <view class="mobile-entry-left">
            <text class="mobile-entry-title">{{ entry.title }}</text>
            <text class="mobile-entry-sub">{{ entry.subtitle }}</text>
          </view>
          <text class="mobile-entry-icon">{{ entry.icon }}</text>
        </view>
      </view>

      <!-- 手机端：分类列表 -->
      <view class="mobile-category">
        <view class="mobile-cat-title">分类</view>
        <view
          v-for="(cat, index) in categories"
          :key="index"
          class="mobile-cat-item"
          :class="{ active: activeCat === index }"
          @tap="onCatTap(index)"
        >
          <text v-if="index === 0" class="mobile-cat-text highlight">券后7.7折起 / 抢省钱补贴</text>
          <text v-else class="mobile-cat-text">{{ cat }}</text>
        </view>
      </view>

    </view>
    <!-- #endif -->

    <!-- ===== PC端：三栏布局（仅H5） ===== -->
    <!-- #ifdef H5 -->
    <view class="pc-body">
      <view class="pc-wrapper">

        <!-- 左侧分类 -->
        <view class="pc-col-left">
          <view class="pc-cat-title">分类</view>
          <view
            v-for="(cat, index) in categories"
            :key="index"
            class="pc-cat-item"
            :class="{ active: activeCat === index }"
            @tap="onCatTap(index)"
          >
            <text v-if="index === 0" class="pc-cat-text highlight">券后7.7折起 / 抢省钱补贴</text>
            <text v-else class="pc-cat-text">{{ cat }}</text>
          </view>
        </view>

        <!-- 中间Banner -->
        <view class="pc-col-center" @mouseenter="isHover = true" @mouseleave="isHover = false">
          <swiper
            class="pc-swiper"
            :indicator-dots="false"
            :autoplay="!isHover"
            :interval="3000"
            :circular="true"
            :current="swiperCurrent"
            @change="onSwiperChange"
          >
            <swiper-item v-for="banner in banners" :key="banner.id">
              <view class="pc-banner-item" :style="{ background: banner.bg }" @tap="onBannerTap(banner)">
                <view class="pc-banner-content">
                  <text class="pc-new-tag">新热周</text>
                  <text class="pc-banner-title">{{ banner.title }}</text>
                  <text class="pc-banner-subtitle">{{ banner.subtitle }}</text>
                  <text class="pc-banner-date">{{ banner.date }}</text>
                </view>
                <view class="pc-banner-images">
                  <view v-for="(img, i) in banner.mockImages" :key="i" class="pc-banner-img" :class="'pbi-' + i">
                    <text class="pc-banner-icon">{{ img.icon }}</text>
                  </view>
                </view>
              </view>
            </swiper-item>
          </swiper>
          <!-- 左右切换按钮 -->
          <view class="swiper-btn prev" v-show="isHover" @tap="prevSlide">〈</view>
          <view class="swiper-btn next" v-show="isHover" @tap="nextSlide">〉</view>
          <!-- 圆点 -->
          <view class="pc-dots">
            <view
              v-for="(b, i) in banners"
              :key="i"
              class="pc-dot"
              :class="{ 'pc-dot-active': swiperCurrent === i }"
              @tap="goToSlide(i)"
            ></view>
          </view>
        </view>

        <!-- 右侧快捷入口 -->
        <view class="pc-col-right">
          <view
            v-for="entry in quickEntries"
            :key="entry.id"
            class="pc-entry-card"
            :style="{ background: entry.color }"
            @tap="onEntryTap(entry)"
          >
            <view class="pc-entry-text">
              <text class="pc-entry-title">{{ entry.title }}</text>
              <text class="pc-entry-sub">{{ entry.subtitle }}</text>
            </view>
            <view class="pc-entry-icon-wrap">
              <text class="pc-entry-icon">{{ entry.icon }}</text>
            </view>
          </view>
        </view>

      </view>
    </view>
    <!-- #endif -->

    <!-- 底部占位 -->
    <view class="tab-placeholder"></view>

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
      touchStartX: 0,

      navItems: [
        { id: 1, name: '国家补贴', icon: '🔰', bg: '#4CAF50' },
        { id: 2, name: '淘宝秒杀', icon: '⏱', bg: '#FF5722' },
        { id: 3, name: 'U先试用', icon: 'U', bg: '#9C27B0' },
        { id: 4, name: '领券中心', icon: '券', bg: '#FF9800' },
        { id: 5, name: '百亿补贴', icon: '补', bg: '#F44336' },
        { id: 6, name: '聚划算', icon: '聚', bg: '#E91E63' },
        { id: 7, name: '天天特卖', icon: '🔥', bg: '#FF5000' },
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

      banners: [
        {
          id: 1, title: '运动户外', subtitle: '春野爆款8.8折起', date: '03.23-03.31火爆热卖',
          bg: '#C58925',
          mockImages: [{ icon: '🛹' }, { icon: '🏈' }, { icon: '📷' }]
        },
        {
          id: 2, title: '数码好物', subtitle: '新品直降5折起', date: '03.23-03.31限时特惠',
          bg: 'linear-gradient(135deg, #1a237e, #3949AB)',
          mockImages: [{ icon: '💻' }, { icon: '📱' }, { icon: '🎧' }]
        },
        {
          id: 3, title: '家居生活', subtitle: '精选爆款7折起', date: '03.23-03.31超值抢购',
          bg: 'linear-gradient(135deg, #2E7D32, #66BB6A)',
          mockImages: [{ icon: '🪴' }, { icon: '🛋️' }, { icon: '💡' }]
        },
        {
          id: 4, title: '美妆护肤', subtitle: '大牌低至3折起', date: '03.23-03.31品质特惠',
          bg: 'linear-gradient(135deg, #880E4F, #E91E63)',
          mockImages: [{ icon: '💄' }, { icon: '🧴' }, { icon: '✨' }]
        },
        {
          id: 5, title: '食品生鲜', subtitle: '新鲜直达9折起', date: '03.23-03.31限时秒杀',
          bg: 'linear-gradient(135deg, #BF360C, #FF7043)',
          mockImages: [{ icon: '🍎' }, { icon: '🥩' }, { icon: '🍜' }]
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
      if (!this.searchKeyword.trim()) {
        uni.showToast({ title: '请输入搜索关键词', icon: 'none' })
        return
      }
      uni.navigateTo({ url: `/pages/search/search?keyword=${encodeURIComponent(this.searchKeyword)}` })
    },
    onHotKwTap(kw) { this.searchKeyword = kw; this.onSearch() },
    onCatTap(index) { this.activeCat = index },
    onBannerTap(banner) { uni.showToast({ title: banner.title, icon: 'none' }) },
    onEntryTap(entry) { uni.showToast({ title: entry.title, icon: 'none' }) },
    onSwiperChange(e) { this.swiperCurrent = e.detail.current },
    prevSlide() { this.swiperCurrent = (this.swiperCurrent - 1 + this.banners.length) % this.banners.length },
    nextSlide() { this.swiperCurrent = (this.swiperCurrent + 1) % this.banners.length },
    goToSlide(index) { this.swiperCurrent = index },
    onTouchStart(e) { this.touchStartX = e.touches[0].clientX },
    onTouchEnd(e) {
      const diff = e.changedTouches[0].clientX - this.touchStartX
      if (diff > 50) this.prevSlide()
      else if (diff < -50) this.nextSlide()
    },
  }
}
</script>

<style lang="scss">
/* ========================================
   全局基础
======================================== */
.page-container {
  min-height: 100vh;
  background: #f5f5f5;
  font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", sans-serif;
}

.status-bar-placeholder {
  background: #ff9ea7;
}

/* ========================================
   第一层：顶部导航广告条（手机+PC通用）
======================================== */
.layer-1-ad {
  position: relative;
  width: 100%;
  background: linear-gradient(90deg, #ff9ea7, #ff5c77);
  display: flex;
  align-items: center;
  min-height: 100rpx;

  .ad-scroll {
    flex: 1;
    white-space: nowrap;
  }

  .ad-nav-list {
    display: inline-flex;
    align-items: center;
    padding: 16rpx 80rpx 16rpx 30rpx;
    gap: 36rpx;
  }

  .ad-nav-item {
    display: inline-flex;
    flex-direction: column;
    align-items: center;
    flex-shrink: 0;
    cursor: pointer;

    .nav-icon-wrap {
      width: 64rpx;
      height: 64rpx;
      border-radius: 16rpx;
      display: flex;
      align-items: center;
      justify-content: center;
      margin-bottom: 6rpx;
      .nav-icon { font-size: 32rpx; }
    }

    .nav-text { font-size: 20rpx; color: #fff; white-space: nowrap; }
  }

  .nav-banner-btn {
    display: inline-flex;
    align-items: center;
    background: linear-gradient(135deg, #FF4D00, #FF8C00);
    border-radius: 12rpx;
    padding: 14rpx 24rpx;
    flex-shrink: 0;
    cursor: pointer;
    .nav-banner-text { font-size: 24rpx; color: #fff; font-weight: bold; white-space: nowrap; }
  }

  .close-btn {
    position: absolute;
    right: 20rpx;
    top: 50%;
    transform: translateY(-50%);
    width: 44rpx;
    height: 44rpx;
    border: 2rpx solid rgba(255,255,255,0.7);
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    color: #fff;
    font-size: 22rpx;
    cursor: pointer;
    background: rgba(0,0,0,0.15);
  }
}

/* ========================================
   第二层：搜索区（手机+PC通用）
======================================== */
.layer-2-header {
  background: #fff;
  padding: 0;

  .header-inner {
    display: flex;
    align-items: center;
    padding: 24rpx 30rpx;
    gap: 20rpx;
    /* #ifdef H5 */
    max-width: 1200px;
    margin: 0 auto;
    padding: 20rpx 0;
    /* #endif */
  }

  .header-left-logo {
    display: flex;
    flex-direction: column;
    align-items: center;
    flex-shrink: 0;
    cursor: pointer;
    .logo-text {
      font-size: 48rpx;
      font-weight: bold;
      color: #FF5000;
      line-height: 1;
      /* #ifdef H5 */
      font-size: 32px;
      /* #endif */
    }
    .logo-sub {
      font-size: 18rpx;
      color: #FF5000;
      margin-top: 4rpx;
      /* #ifdef H5 */
      font-size: 12px;
      /* #endif */
    }
  }

  .header-right-search {
    flex: 1;
    display: flex;
    flex-direction: column;
    gap: 12rpx;

    .search-box {
      display: flex;
      align-items: center;
      border: 3rpx solid #FF5000;
      border-radius: 60rpx;
      height: 72rpx;
      overflow: hidden;
      /* #ifdef H5 */
      border-width: 2px;
      height: 42px;
      max-width: 860px;
      /* #endif */

      .search-type-wrap {
        display: flex;
        align-items: center;
        padding: 0 20rpx;
        height: 100%;
        background: #f5f5f5;
        border-right: 2rpx solid #eee;
        flex-shrink: 0;
        cursor: pointer;
        .search-type-text { font-size: 26rpx; color: #333; }
        .search-type-arrow { font-size: 20rpx; color: #999; margin-left: 6rpx; }
      }

      .search-divider { width: 1px; height: 60%; background: #eee; flex-shrink: 0; }

      .search-input {
        flex: 1;
        height: 100%;
        padding: 0 20rpx;
        font-size: 26rpx;
        background: transparent;
        border: none;
        outline: none;
      }
      .search-placeholder { color: #bbb; font-size: 26rpx; }

      .search-btn {
        background: #FF5000;
        height: 100%;
        padding: 0 40rpx;
        display: flex;
        align-items: center;
        justify-content: center;
        flex-shrink: 0;
        cursor: pointer;
        &:hover { background: #e64500; }
        .search-btn-text {
          color: #fff;
          font-size: 28rpx;
          font-weight: bold;
          letter-spacing: 4rpx;
        }
      }
    }

    .hot-keywords {
      display: flex;
      flex-wrap: wrap;
      gap: 12rpx 24rpx;
      .hot-kw-item {
        font-size: 22rpx;
        color: #999;
        cursor: pointer;
        white-space: nowrap;
        &:hover { color: #FF5000; }
      }
    }
  }
}

/* ========================================
   手机端主体（仅App/小程序，#ifndef H5）
======================================== */
.mobile-body {
  display: flex;
  flex-direction: column;
}

/* 手机Banner */
.mobile-banner-wrap {
  position: relative;
  width: 100%;

  .mobile-swiper {
    width: 100%;
    height: 360rpx;

    .mobile-banner-item {
      width: 100%;
      height: 100%;
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 40rpx 30rpx;
      box-sizing: border-box;

      .mobile-banner-content {
        display: flex;
        flex-direction: column;
        flex: 1;

        .mobile-new-tag {
          background: #FF5000;
          color: #fff;
          font-size: 20rpx;
          padding: 4rpx 14rpx;
          border-radius: 6rpx;
          align-self: flex-start;
          margin-bottom: 16rpx;
        }
        .mobile-banner-title {
          font-size: 44rpx;
          font-weight: bold;
          color: #fff;
          margin-bottom: 8rpx;
        }
        .mobile-banner-subtitle {
          font-size: 28rpx;
          color: #fff;
          margin-bottom: 16rpx;
        }
        .mobile-banner-date {
          font-size: 20rpx;
          color: rgba(255,255,255,0.85);
        }
      }

      .mobile-banner-icons {
        position: relative;
        width: 220rpx;
        height: 100%;
        flex-shrink: 0;

        .mobile-banner-icon {
          position: absolute;
          display: flex;
          align-items: center;
          justify-content: center;
          font-size: 70rpx;

          &.mbi-0 { bottom: 20rpx; left: 0; font-size: 60rpx; transform: rotate(-15deg); }
          &.mbi-1 { top: 50%; left: 50%; transform: translate(-50%, -50%); font-size: 110rpx; }
          &.mbi-2 { bottom: 30rpx; right: 0; font-size: 60rpx; transform: rotate(10deg); }
        }
      }
    }
  }

  /* 手机圆点 */
  .mobile-dots {
    position: absolute;
    bottom: 14rpx;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 10rpx;
    z-index: 10;

    .mobile-dot {
      width: 12rpx;
      height: 12rpx;
      border-radius: 50%;
      background: rgba(255,255,255,0.5);
      transition: all 0.3s;
      cursor: pointer;

      &.mobile-dot-active {
        width: 32rpx;
        border-radius: 6rpx;
        background: #fff;
      }
    }
  }
}

/* 手机快捷入口 2x2 */
.mobile-entries {
  display: flex;
  flex-wrap: wrap;
  gap: 16rpx;
  padding: 16rpx;
  background: #f5f5f5;

  .mobile-entry-card {
    width: calc(50% - 8rpx);
    height: 160rpx;
    border-radius: 14rpx;
    padding: 24rpx 20rpx;
    display: flex;
    align-items: center;
    justify-content: space-between;
    box-sizing: border-box;
    cursor: pointer;

    .mobile-entry-left {
      display: flex;
      flex-direction: column;
      .mobile-entry-title { font-size: 30rpx; font-weight: bold; color: #fff; }
      .mobile-entry-sub { font-size: 20rpx; color: rgba(255,255,255,0.9); margin-top: 6rpx; }
    }

    .mobile-entry-icon { font-size: 52rpx; }
  }
}

/* 手机分类列表 */
.mobile-category {
  background: #fff;
  margin-top: 16rpx;
  padding: 0 0 30rpx;

  .mobile-cat-title {
    font-size: 30rpx;
    font-weight: bold;
    color: #333;
    padding: 24rpx 30rpx 16rpx;
    border-bottom: 1rpx solid #f0f0f0;
  }

  .mobile-cat-item {
    padding: 26rpx 30rpx;
    border-bottom: 1rpx solid #f5f5f5;
    border-left: 6rpx solid transparent;
    cursor: pointer;

    &.active {
      border-left-color: #FF5000;
      background: #fff5f0;
      .mobile-cat-text { color: #FF5000; font-weight: bold; }
    }

    .mobile-cat-text { font-size: 28rpx; color: #333; display: block; }
    .highlight { color: #FF5000; font-weight: bold; }
  }
}

/* ========================================
   PC端主体（仅H5）
======================================== */
.pc-body {
  margin-top: 12px;

  .pc-wrapper {
    display: flex;
    align-items: stretch;
    max-width: 1200px;
    margin: 0 auto;
    height: 420px;
    gap: 10px;
  }
}

/* PC左侧分类 */
.pc-col-left {
  width: 150px;
  min-width: 150px;
  flex-shrink: 0;
  background: #f7f9fa;
  border-radius: 10px;
  overflow-y: auto;
  padding-bottom: 10px;

  .pc-cat-title {
    font-size: 15px;
    font-weight: bold;
    color: #333;
    padding: 14px 14px 10px;
    border-bottom: 1px solid #eee;
  }

  .pc-cat-item {
    padding: 10px 14px;
    border-left: 3px solid transparent;
    cursor: pointer;

    &:hover { background: #ffebe5; .pc-cat-text { color: #FF5000; } }
    &.active {
      border-left-color: #FF5000;
      background: #fff5f0;
      .pc-cat-text { color: #FF5000; font-weight: bold; }
    }

    .pc-cat-text {
      font-size: 12px;
      color: #555;
      display: block;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }
    .highlight { color: #FF5000; font-weight: bold; }
  }
}

/* PC中间Banner */
.pc-col-center {
  flex: 1;
  min-width: 0;
  border-radius: 12px;
  overflow: hidden;
  position: relative;
  height: 100%;

  .pc-swiper {
    width: 100%;
    height: 100%;

    .pc-banner-item {
      width: 100%;
      height: 100%;
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 40px;
      box-sizing: border-box;
      cursor: pointer;

      .pc-banner-content {
        display: flex;
        flex-direction: column;
        flex-shrink: 0;

        .pc-new-tag {
          background: #FF5000;
          color: #fff;
          font-size: 12px;
          padding: 3px 10px;
          border-radius: 5px;
          align-self: flex-start;
          margin-bottom: 14px;
        }
        .pc-banner-title { font-size: 32px; font-weight: bold; color: #fff; margin-bottom: 8px; }
        .pc-banner-subtitle { font-size: 20px; color: #fff; margin-bottom: 16px; }
        .pc-banner-date { font-size: 13px; color: rgba(255,255,255,0.85); }
      }

      .pc-banner-images {
        position: relative;
        width: 260px;
        height: 100%;
        flex-shrink: 0;

        .pc-banner-img {
          position: absolute;
          display: flex;
          align-items: center;
          justify-content: center;

          .pc-banner-icon { display: block; }

          &.pbi-0 { bottom: 16px; left: 0; .pc-banner-icon { font-size: 64px; transform: rotate(-15deg); } }
          &.pbi-1 { top: 50%; left: 70px; transform: translateY(-50%); .pc-banner-icon { font-size: 110px; } }
          &.pbi-2 { bottom: 20px; right: 0; .pc-banner-icon { font-size: 64px; transform: rotate(10deg); } }
        }
      }
    }
  }

  /* 左右按钮 */
  .swiper-btn {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    width: 36px;
    height: 36px;
    background: rgba(0,0,0,0.35);
    color: #fff;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 16px;
    cursor: pointer;
    z-index: 20;
    transition: background 0.2s;
    user-select: none;
    &:hover { background: rgba(0,0,0,0.6); }
    &.prev { left: 10px; }
    &.next { right: 10px; }
  }

  /* PC圆点 */
  .pc-dots {
    position: absolute;
    bottom: 12px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 6px;
    z-index: 20;

    .pc-dot {
      width: 8px;
      height: 8px;
      border-radius: 50%;
      background: rgba(255,255,255,0.5);
      cursor: pointer;
      transition: all 0.3s;

      &.pc-dot-active {
        width: 20px;
        border-radius: 4px;
        background: #fff;
      }
    }
  }
}

/* PC右侧快捷入口 */
.pc-col-right {
  width: 155px;
  min-width: 155px;
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
  gap: 8px;
  height: 100%;

  .pc-entry-card {
    flex: 1;
    border-radius: 10px;
    padding: 0 14px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    cursor: pointer;
    color: #fff;
    transition: filter 0.2s;
    &:hover { filter: brightness(1.08); }

    .pc-entry-text {
      display: flex;
      flex-direction: column;
      .pc-entry-title { font-size: 15px; font-weight: bold; margin-bottom: 2px; }
      .pc-entry-sub { font-size: 11px; opacity: 0.9; }
    }

    .pc-entry-icon-wrap {
      width: 42px;
      height: 42px;
      background: rgba(255,255,255,0.2);
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      .pc-entry-icon { font-size: 20px; }
    }
  }
}

/* 底部占位 */
.tab-placeholder { height: 100rpx; }
</style>