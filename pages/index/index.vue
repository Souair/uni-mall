<template>
  <view class="page-container">

    <view :style="{ height: statusBarHeight + 'px' }" class="status-bar-placeholder"></view>
    <view class="layer-1-ad" v-if="showTopAd">
      <scroll-view scroll-x class="ad-scroll" :show-scrollbar="false">
        <view class="ad-nav-list">
          <view class="ad-nav-item" v-for="item in navItems" :key="item.id" @tap="onNavTap(item)">
            <view class="nav-icon-wrap" :style="{ background: item.bg }"><text class="nav-icon">{{ item.icon }}</text></view>
            <text class="nav-text">{{ item.name }}</text>
          </view>
          <view class="nav-banner-btn" @tap="onNavTap({ name: '百亿补贴' })"><text class="nav-banner-text">百亿补贴来啦</text></view>
        </view>
      </scroll-view>
      <view class="close-btn" @tap="closeTopAd">✕</view>
    </view>

    <view class="layer-2-header">
      <view class="header-inner">
        <view class="header-left-logo"><text class="logo-text">淘宝</text><text class="logo-sub">Taobao.com</text></view>
        <view class="header-right-search">
          <view class="search-box">
            <view class="search-type-wrap" @tap="toggleSearchType"><text class="search-type-text">{{ searchType }}</text><text class="search-type-arrow">▾</text></view>
            <view class="search-divider"></view>
            <input class="search-input" v-model="searchKeyword" :placeholder="currentPlaceholder" placeholder-class="search-placeholder" @confirm="onSearch" />
            <view class="search-btn" @tap="onSearch"><text class="search-btn-text">搜索</text></view>
          </view>
          <view class="hot-keywords">
            <text class="hot-kw-item" v-for="(kw, index) in hotKeywords" :key="index" @tap="onHotKwTap(kw)">{{ kw }}</text>
          </view>
        </view>
      </view>
    </view>

    <view class="mobile-layout-wrapper">
      <view class="mobile-body">
        
        <view class="mobile-banner-wrap">
          <swiper
            class="mobile-swiper"
            :indicator-dots="false"
            :autoplay="true"
            :interval="3000"
            :circular="true"
            :current="mobileSwiperCurrent" 
            @change="onMobileSwiperChange"
          >
            <swiper-item v-for="banner in banners" :key="banner.id">
              <view class="mobile-banner-item" :style="{ background: banner.bg }" @tap="onBannerTap(banner)">
                <view class="mobile-banner-content">
                  <text class="mobile-new-tag">新热周</text><text class="mobile-banner-title">{{ banner.title }}</text><text class="mobile-banner-subtitle">{{ banner.subtitle }}</text><text class="mobile-banner-date">{{ banner.date }}</text>
                </view>
                <view class="mobile-banner-icons">
                  <view v-for="(img, i) in banner.mockImages" :key="i" class="mobile-banner-icon" :class="'mbi-' + i"><text>{{ img.icon }}</text></view>
                </view>
              </view>
            </swiper-item>
          </swiper>
          <view class="mobile-dots">
            <view
              v-for="(b, i) in banners"
              :key="i"
              class="mobile-dot"
              :class="{ 'mobile-dot-active': mobileSwiperCurrent === i }"
              @tap="goToMobileSlide(i)"
            ></view>
          </view>
        </view>

        <view class="mobile-entries">
          <view v-for="entry in quickEntries" :key="entry.id" class="mobile-entry-card" :style="{ background: entry.color }" @tap="onEntryTap(entry)">
            <view class="mobile-entry-left"><text class="mobile-entry-title">{{ entry.title }}</text><text class="mobile-entry-sub">{{ entry.subtitle }}</text></view>
            <text class="mobile-entry-icon">{{ entry.icon }}</text>
          </view>
        </view>

        <view class="mobile-category">
          <view class="mobile-cat-title">分类</view>
          <view v-for="(cat, index) in categories" :key="index" class="mobile-cat-item-wrap">
            <view class="mobile-cat-item" :class="{ active: activeMobileCat === index }" @tap="toggleMobileCat(index)">
              <text v-if="index === 0" class="mobile-cat-text highlight">{{ cat.name }}</text>
              <text v-else class="mobile-cat-text">{{ cat.name }}</text>
              <text class="mobile-cat-arrow" v-if="cat.subCategories">{{ activeMobileCat === index ? '▲' : '▼' }}</text>
            </view>
            <view class="mobile-sub-menu" v-show="activeMobileCat === index && cat.subCategories">
              <view class="m-sub-group" v-for="(sub, sIndex) in cat.subCategories" :key="sIndex">
                <view class="m-sub-title">{{ sub.group }}</view>
                <view class="m-sub-items">
                  <text class="m-sub-item" v-for="(item, iIndex) in sub.items" :key="iIndex">{{ item }}</text>
                </view>
              </view>
            </view>
          </view>
        </view>

      </view>
    </view>

    <view class="pc-layout-wrapper">
      <view class="pc-body">
        <view class="pc-wrapper" @mouseleave="hoverCatIndex = -1">
          <view class="pc-col-left">
            <view class="pc-cat-title">分类</view>
            <view v-for="(cat, index) in categories" :key="index" class="pc-cat-item" :class="{ active: hoverCatIndex === index }" @mouseenter="onCatHover(index)" @tap="onCatTap(index)">
              <text v-if="index === 0" class="pc-cat-text highlight">{{ cat.name }}</text>
              <text v-else class="pc-cat-text">{{ cat.name }}</text>
              <text class="pc-cat-arrow" v-if="cat.subCategories">&gt;</text>
            </view>
          </view>
          <view class="pc-mega-menu" v-show="hoverCatIndex !== -1 && categories[hoverCatIndex] && categories[hoverCatIndex].subCategories">
            <block v-if="hoverCatIndex !== -1 && categories[hoverCatIndex] && categories[hoverCatIndex].subCategories">
              <view class="mega-group" v-for="(sub, sIndex) in categories[hoverCatIndex].subCategories" :key="sIndex">
                <view class="mega-group-title">{{ sub.group }} &gt;</view>
                <view class="mega-group-items"><text class="mega-item" v-for="(item, iIndex) in sub.items" :key="iIndex">{{ item }}</text></view>
              </view>
            </block>
          </view>

          <view class="pc-col-center" @mouseenter="isHover = true" @mouseleave="isHover = false">
            <swiper
              class="pc-swiper"
              :indicator-dots="false"
              :autoplay="!isHover && hoverCatIndex === -1" 
              :interval="3000"
              :circular="true"
              :current="pcSwiperCurrent" 
              @change="onPcSwiperChange"
            >
              <swiper-item v-for="banner in banners" :key="banner.id">
                <view class="pc-banner-item" :style="{ background: banner.bg }" @tap="onBannerTap(banner)">
                  <view class="pc-banner-content">
                    <text class="pc-new-tag">新热周</text><text class="pc-banner-title">{{ banner.title }}</text><text class="pc-banner-subtitle">{{ banner.subtitle }}</text><text class="pc-banner-date">{{ banner.date }}</text>
                  </view>
                  <view class="pc-banner-images">
                    <view v-for="(img, i) in banner.mockImages" :key="i" class="pc-banner-img" :class="'pbi-' + i"><text class="pc-banner-icon">{{ img.icon }}</text></view>
                  </view>
                </view>
              </swiper-item>
            </swiper>
            <view class="swiper-btn prev" v-show="isHover" @tap="prevPcSlide">〈</view>
            <view class="swiper-btn next" v-show="isHover" @tap="nextPcSlide">〉</view>
            <view class="pc-dots">
              <view
                v-for="(b, i) in banners"
                :key="i"
                class="pc-dot"
                :class="{ 'pc-dot-active': pcSwiperCurrent === i }"
                @tap="goToPcSlide(i)"
              ></view>
            </view>
          </view>

          <view class="pc-col-right">
            <view v-for="entry in quickEntries" :key="entry.id" class="pc-entry-card" :style="{ background: entry.color }" @tap="onEntryTap(entry)">
              <view class="pc-entry-text"><text class="pc-entry-title">{{ entry.title }}</text><text class="pc-entry-sub">{{ entry.subtitle }}</text></view>
              <view class="pc-entry-icon-wrap"><text class="pc-entry-icon">{{ entry.icon }}</text></view>
            </view>
          </view>

        </view>
      </view>
    </view>

    <view class="tab-placeholder"></view>
  </view>
</template>

<script>
export default {
  name: 'IndexPage',
  data() {
    return {
      isHover: false,
      statusBarHeight: 0,
      showTopAd: true,
      searchKeyword: '',
      searchType: '宝贝',
      currentPlaceholder: '第一人称拍摄设备',
      
      // ✅ 修复点 4：彻底分开两端的变量
      mobileSwiperCurrent: 0, 
      pcSwiperCurrent: 0,     
      
      activeMobileCat: -1, // 手机端展开状态
      hoverCatIndex: -1,   // PC端悬停状态

      // 下面的数据部分一字未动
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
        { name: '券后7.7折起 / 抢省钱补贴', subCategories: null },
        { 
          name: '电脑 / 配件 / 办公 / 文具', 
          subCategories: [
            { group: '电脑整机', items: ['笔记本电脑', '游戏本', '平板电脑', 'DIY电脑', '服务器/工作站', '一体机'] },
            { group: '电脑配件', items: ['显示器', 'CPU', '主板', '显卡', '硬盘', '内存', '机箱', '电源', '散热器'] },
            { group: '外设产品', items: ['鼠标', '键盘', 'U盘', '移动硬盘', '摄像头', '手写板', 'UPS电源'] },
            { group: '网络产品', items: ['路由器', '网络机顶盒', '交换机', '网卡', '5G/4G上网', '网线'] },
            { group: '办公设备', items: ['投影机', '打印机', '传真设备', '碎纸机', '考勤门禁', '收银机', '监控'] }
          ]
        },
        { 
          name: '家电 / 手机 / 通信 / 数码', 
          subCategories: [
            { group: '热门手机', items: ['Apple', '华为', '小米', '荣耀', 'OPPO', 'vivo', '三星'] },
            { group: '大家电', items: ['电视', '空调', '冰箱', '洗衣机', '家庭影院', '酒柜'] },
            { group: '生活电器', items: ['电风扇', '冷风扇', '吸尘器', '空气净化器', '扫地机器人', '除湿机'] }
          ]
        },
        { name: '工业品 / 商业 / 农业 / 定制', subCategories: null },
        { name: '家具 / 家装 / 家居 / 厨具', subCategories: null },
        { name: '女装 / 男装 / 内衣 / 配饰', subCategories: null },
        { name: '女鞋 / 男鞋 / 运动 / 户外', subCategories: null },
        { name: '汽车 / 珠宝 / 文玩 / 箱包', subCategories: null },
        { name: '食品 / 鲜花 / 酒水 / 健康', subCategories: null },
        { name: '母婴 / 童装 / 玩具 / 宠物', subCategories: null },
        { name: '美妆 / 个护 / 娱乐 / 图书', subCategories: null },
      ],
      banners: [
        {
          id: 1, title: '运动户外', subtitle: '春野爆款8.8折起', date: '03.23-03.31火爆热卖',
          bg: '#C58925', mockImages: [{ icon: '🛹' }, { icon: '🏈' }, { icon: '📷' }]
        },
        {
          id: 2, title: '数码好物', subtitle: '新品直降5折起', date: '03.23-03.31限时特惠',
          bg: 'linear-gradient(135deg, #1a237e, #3949AB)', mockImages: [{ icon: '💻' }, { icon: '📱' }, { icon: '🎧' }]
        },
        {
          id: 3, title: '家居生活', subtitle: '精选爆款7折起', date: '03.23-03.31超值抢购',
          bg: 'linear-gradient(135deg, #2E7D32, #66BB6A)', mockImages: [{ icon: '🪴' }, { icon: '🛋️' }, { icon: '💡' }]
        },
        {
          id: 4, title: '美妆护肤', subtitle: '大牌低至3折起', date: '03.23-03.31品质特惠',
          bg: 'linear-gradient(135deg, #880E4F, #E91E63)', mockImages: [{ icon: '💄' }, { icon: '🧴' }, { icon: '✨' }]
        },
        {
          id: 5, title: '食品生鲜', subtitle: '新鲜直达9折起', date: '03.23-03.31限时秒杀',
          bg: 'linear-gradient(135deg, #BF360C, #FF7043)', mockImages: [{ icon: '🍎' }, { icon: '🥩' }, { icon: '🍜' }]
        }
      ],
      quickEntries: [
        { id: 1, title: '品质家居', subtitle: '超值优惠', icon: '🛋️', color: '#FF5722' },
        { id: 2, title: '精致美妆', subtitle: '品质之选', icon: '💄', color: '#E91E63' },
        { id: 3, title: '品质五金', subtitle: '超值特惠', icon: '🧰', color: '#FF9800' },
        { id: 4, title: '超值百货', subtitle: '省钱省心', icon: '🧴', color: '#03A9F4' }
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
    
    toggleMobileCat(index) { this.activeMobileCat = this.activeMobileCat === index ? -1 : index; },
    onCatHover(index) { this.hoverCatIndex = index; },
    onCatTap(index) { uni.showToast({ title: this.categories[index].name, icon: 'none' }); },
    onBannerTap(banner) { uni.showToast({ title: banner.title, icon: 'none' }) },
    onEntryTap(entry) { uni.showToast({ title: entry.title, icon: 'none' }) },

    // ✅ 修复点 5：分别处理手机端和PC端的轮播逻辑
    onMobileSwiperChange(e) { this.mobileSwiperCurrent = e.detail.current; },
    goToMobileSlide(index) { this.mobileSwiperCurrent = index; },

    onPcSwiperChange(e) { this.pcSwiperCurrent = e.detail.current; },
    prevPcSlide() { this.pcSwiperCurrent = (this.pcSwiperCurrent - 1 + this.banners.length) % this.banners.length; },
    nextPcSlide() { this.pcSwiperCurrent = (this.pcSwiperCurrent + 1) % this.banners.length; },
    goToPcSlide(index) { this.pcSwiperCurrent = index; }
    
    // (已删除原本冲突的 onTouchStart 和 onTouchEnd)
  }
}
</script>
<style lang="scss">
/* ========================================
   全局基础与响应式控制
======================================== */
.page-container {
  min-height: 100vh;
  background: #f5f5f5;
  font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", sans-serif;
}

.status-bar-placeholder {
  background: #ff9ea7;
}

/* 默认隐藏PC端，显示手机端 */
.mobile-layout-wrapper { display: block; }
.pc-layout-wrapper { display: none; }

@media screen and (min-width: 768px) {
  .mobile-layout-wrapper { display: none; }
  .pc-layout-wrapper { display: block; }
}

/* ========================================
   第一层/第二层：通用广告与搜索
======================================== */
.layer-1-ad {
  position: relative; width: 100%; background: linear-gradient(90deg, #ff9ea7, #ff5c77);
  display: flex; align-items: center; min-height: 100rpx;
  .ad-scroll { flex: 1; white-space: nowrap; }
  .ad-nav-list { display: inline-flex; align-items: center; padding: 16rpx 80rpx 16rpx 30rpx; gap: 36rpx; }
  .ad-nav-item {
    display: inline-flex; flex-direction: column; align-items: center; flex-shrink: 0; cursor: pointer;
    .nav-icon-wrap { width: 64rpx; height: 64rpx; border-radius: 16rpx; display: flex; align-items: center; justify-content: center; margin-bottom: 6rpx; .nav-icon { font-size: 32rpx; } }
    .nav-text { font-size: 20rpx; color: #fff; white-space: nowrap; }
  }
  .nav-banner-btn {
    display: inline-flex; align-items: center; background: linear-gradient(135deg, #FF4D00, #FF8C00); border-radius: 12rpx; padding: 14rpx 24rpx; flex-shrink: 0; cursor: pointer;
    .nav-banner-text { font-size: 24rpx; color: #fff; font-weight: bold; white-space: nowrap; }
  }
  .close-btn { position: absolute; right: 20rpx; top: 50%; transform: translateY(-50%); width: 44rpx; height: 44rpx; border: 2rpx solid rgba(255,255,255,0.7); border-radius: 50%; display: flex; align-items: center; justify-content: center; color: #fff; font-size: 22rpx; cursor: pointer; background: rgba(0,0,0,0.15); }
}

.layer-2-header {
  background: #fff; padding: 0;
  .header-inner {
    display: flex; align-items: center; padding: 24rpx 30rpx; gap: 20rpx; max-width: 1200px; margin: 0 auto; 
    @media screen and (min-width: 768px) { padding: 20px 0; }
  }
  .header-left-logo {
    display: flex; flex-direction: column; align-items: center; flex-shrink: 0; cursor: pointer;
    .logo-text { font-size: 48rpx; font-weight: bold; color: #FF5000; line-height: 1; @media screen and (min-width: 768px) { font-size: 32px; } }
    .logo-sub { font-size: 18rpx; color: #FF5000; margin-top: 4rpx; @media screen and (min-width: 768px) { font-size: 12px; } }
  }
  .header-right-search {
    flex: 1; display: flex; flex-direction: column; gap: 12rpx;
    .search-box {
      display: flex; align-items: center; border: 3rpx solid #FF5000; border-radius: 60rpx; height: 72rpx; overflow: hidden; 
      @media screen and (min-width: 768px) { border-width: 2px; height: 42px; max-width: 860px; }
      .search-type-wrap { display: flex; align-items: center; padding: 0 20rpx; height: 100%; background: #f5f5f5; border-right: 2rpx solid #eee; flex-shrink: 0; cursor: pointer; .search-type-text { font-size: 26rpx; color: #333; } .search-type-arrow { font-size: 20rpx; color: #999; margin-left: 6rpx; } }
      .search-divider { width: 1px; height: 60%; background: #eee; flex-shrink: 0; }
      .search-input { flex: 1; height: 100%; padding: 0 20rpx; font-size: 26rpx; background: transparent; border: none; outline: none; }
      .search-placeholder { color: #bbb; font-size: 26rpx; }
      .search-btn { background: #FF5000; height: 100%; padding: 0 40rpx; display: flex; align-items: center; justify-content: center; flex-shrink: 0; cursor: pointer; &:hover { background: #e64500; } .search-btn-text { color: #fff; font-size: 28rpx; font-weight: bold; letter-spacing: 4rpx; } }
    }
    .hot-keywords {
      display: flex; flex-wrap: wrap; gap: 12rpx 24rpx;
      .hot-kw-item { font-size: 22rpx; color: #999; cursor: pointer; white-space: nowrap; &:hover { color: #FF5000; } }
    }
  }
}

/* ========================================
   手机端主体样式
======================================== */
.mobile-body { display: flex; flex-direction: column; }
.mobile-banner-wrap {
  position: relative; width: 100%;
  .mobile-swiper {
    width: 100%; height: 360rpx;
    .mobile-banner-item {
      width: 100%; height: 100%; display: flex; align-items: center; justify-content: space-between; padding: 40rpx 30rpx; box-sizing: border-box;
      .mobile-banner-content { display: flex; flex-direction: column; flex: 1; .mobile-new-tag { background: #FF5000; color: #fff; font-size: 20rpx; padding: 4rpx 14rpx; border-radius: 6rpx; align-self: flex-start; margin-bottom: 16rpx; } .mobile-banner-title { font-size: 44rpx; font-weight: bold; color: #fff; margin-bottom: 8rpx; } .mobile-banner-subtitle { font-size: 28rpx; color: #fff; margin-bottom: 16rpx; } .mobile-banner-date { font-size: 20rpx; color: rgba(255,255,255,0.85); } }
      .mobile-banner-icons { position: relative; width: 220rpx; height: 100%; flex-shrink: 0; .mobile-banner-icon { position: absolute; display: flex; align-items: center; justify-content: center; font-size: 70rpx; &.mbi-0 { bottom: 20rpx; left: 0; font-size: 60rpx; transform: rotate(-15deg); } &.mbi-1 { top: 50%; left: 50%; transform: translate(-50%, -50%); font-size: 110rpx; } &.mbi-2 { bottom: 30rpx; right: 0; font-size: 60rpx; transform: rotate(10deg); } } }
    }
  }
  .mobile-dots { position: absolute; bottom: 14rpx; left: 50%; transform: translateX(-50%); display: flex; gap: 10rpx; z-index: 10; .mobile-dot { width: 12rpx; height: 12rpx; border-radius: 50%; background: rgba(255,255,255,0.5); transition: all 0.3s; cursor: pointer; &.mobile-dot-active { width: 32rpx; border-radius: 6rpx; background: #fff; } } }
}

.mobile-entries {
  display: flex; flex-wrap: wrap; gap: 16rpx; padding: 16rpx; background: #f5f5f5;
  .mobile-entry-card { width: calc(50% - 8rpx); height: 160rpx; border-radius: 14rpx; padding: 24rpx 20rpx; display: flex; align-items: center; justify-content: space-between; box-sizing: border-box; cursor: pointer; .mobile-entry-left { display: flex; flex-direction: column; .mobile-entry-title { font-size: 30rpx; font-weight: bold; color: #fff; } .mobile-entry-sub { font-size: 20rpx; color: rgba(255,255,255,0.9); margin-top: 6rpx; } } .mobile-entry-icon { font-size: 52rpx; } }
}

/* 🌟 手机分类：手风琴 */
.mobile-category {
  background: #fff; margin-top: 16rpx; padding: 0 0 30rpx;
  .mobile-cat-title { font-size: 30rpx; font-weight: bold; color: #333; padding: 24rpx 30rpx 16rpx; border-bottom: 1rpx solid #f0f0f0; }
  .mobile-cat-item-wrap { border-bottom: 1rpx solid #f5f5f5; }
  .mobile-cat-item {
    padding: 26rpx 30rpx; border-left: 6rpx solid transparent; display: flex; justify-content: space-between; align-items: center; cursor: pointer;
    &.active { border-left-color: #FF5000; background: #fff5f0; .mobile-cat-text { color: #FF5000; font-weight: bold; } }
    .mobile-cat-text { font-size: 28rpx; color: #333; }
    .mobile-cat-arrow { font-size: 24rpx; color: #999; }
    .highlight { color: #FF5000; font-weight: bold; }
  }
  .mobile-sub-menu {
    background: #fafafa; padding: 20rpx 30rpx;
    .m-sub-group { margin-bottom: 20rpx; &:last-child { margin-bottom: 0; } }
    .m-sub-title { font-size: 26rpx; font-weight: bold; color: #333; margin-bottom: 12rpx; }
    .m-sub-items { display: flex; flex-wrap: wrap; gap: 16rpx; }
    .m-sub-item { font-size: 24rpx; color: #666; background: #fff; padding: 8rpx 20rpx; border-radius: 6rpx; border: 1px solid #eee; }
  }
}

/* ========================================
   PC端主体样式
======================================== */
.pc-body {
  margin-top: 12px;
  .pc-wrapper { position: relative; display: flex; align-items: stretch; max-width: 1200px; margin: 0 auto; height: 510px; gap: 10px; }
}

.pc-col-left {
  width: 220px; min-width: 220px; flex-shrink: 0; background: #f7f9fa; border-radius: 10px; overflow-y: auto; padding-bottom: 10px;
  .pc-cat-title { font-size: 15px; font-weight: bold; color: #333; padding: 14px 14px 10px; border-bottom: 1px solid #eee; }
  .pc-cat-item {
    padding: 12px 14px; border-left: 3px solid transparent; cursor: pointer; display: flex; justify-content: space-between; align-items: center;
    &:hover, &.active { background: #ffebe5; .pc-cat-text { color: #FF5000; font-weight: bold; } .pc-cat-arrow { color: #FF5000; } }
    .pc-cat-text { font-size: 13px; color: #555; display: block; white-space: nowrap; overflow: visible; text-overflow: unset; }
    .pc-cat-arrow { font-size: 12px; color: #ccc; }
    .highlight { color: #FF5000; font-weight: bold; }
  }
}

/* 🌟 PC端：悬停超级菜单 */
.pc-mega-menu {
  position: absolute; left: 220px; top: 0; height: 100%; width: 780px; background: #fff; z-index: 999; box-shadow: 4px 0 10px rgba(0,0,0,0.05); padding: 20px 30px; box-sizing: border-box; overflow-y: auto;
  .mega-group { display: flex; align-items: flex-start; margin-bottom: 20px; border-bottom: 1px dashed #eee; padding-bottom: 15px; &:last-child { border-bottom: none; } }
  .mega-group-title { width: 90px; font-size: 14px; font-weight: bold; color: #333; flex-shrink: 0; margin-top: 2px; cursor: pointer; &:hover { color: #FF5000; } }
  .mega-group-items { flex: 1; display: flex; flex-wrap: wrap; gap: 12px 20px; }
  .mega-item { font-size: 13px; color: #666; cursor: pointer; white-space: nowrap; &:hover { color: #FF5000; } }
}

.pc-col-center {
  flex: 1; min-width: 0; border-radius: 12px; overflow: hidden; position: relative; height: 100%;
  .pc-swiper {
    width: 100%; height: 100%;
    .pc-banner-item {
      width: 100%; height: 100%; display: flex; align-items: center; justify-content: space-between; padding: 40px; box-sizing: border-box; cursor: pointer;
      .pc-banner-content { display: flex; flex-direction: column; flex-shrink: 0; .pc-new-tag { background: #FF5000; color: #fff; font-size: 12px; padding: 3px 10px; border-radius: 5px; align-self: flex-start; margin-bottom: 18px; } .pc-banner-title { font-size: 36px; font-weight: bold; color: #fff; margin-bottom: 12px; } .pc-banner-subtitle { font-size: 22px; color: #fff; margin-bottom: 24px; } .pc-banner-date { font-size: 14px; color: rgba(255,255,255,0.85); } }
      .pc-banner-images { position: relative; width: 260px; height: 100%; flex-shrink: 0; .pc-banner-img { position: absolute; display: flex; align-items: center; justify-content: center; .pc-banner-icon { display: block; } &.pbi-0 { bottom: 24px; left: 0; .pc-banner-icon { font-size: 70px; transform: rotate(-15deg); } } &.pbi-1 { top: 50%; left: 60px; transform: translateY(-50%); .pc-banner-icon { font-size: 130px; } } &.pbi-2 { bottom: 30px; right: 0; .pc-banner-icon { font-size: 70px; transform: rotate(10deg); } } } }
    }
  }
  .swiper-btn { position: absolute; top: 50%; transform: translateY(-50%); width: 36px; height: 36px; background: rgba(0,0,0,0.35); color: #fff; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 16px; cursor: pointer; z-index: 20; transition: background 0.2s; user-select: none; &:hover { background: rgba(0,0,0,0.6); } &.prev { left: 10px; } &.next { right: 10px; } }
  .pc-dots { position: absolute; bottom: 12px; left: 50%; transform: translateX(-50%); display: flex; gap: 6px; z-index: 20; .pc-dot { width: 8px; height: 8px; border-radius: 50%; background: rgba(255,255,255,0.5); cursor: pointer; transition: all 0.3s; &.pc-dot-active { width: 20px; border-radius: 4px; background: #fff; } } }
}

.pc-col-right {
  width: 155px; min-width: 155px; flex-shrink: 0; display: flex; flex-direction: column; gap: 8px; height: 100%;
  .pc-entry-card { flex: 1; border-radius: 10px; padding: 0 14px; display: flex; align-items: center; justify-content: space-between; cursor: pointer; color: #fff; transition: filter 0.2s; &:hover { filter: brightness(1.08); } .pc-entry-text { display: flex; flex-direction: column; .pc-entry-title { font-size: 15px; font-weight: bold; margin-bottom: 2px; } .pc-entry-sub { font-size: 11px; opacity: 0.9; } } .pc-entry-icon-wrap { width: 42px; height: 42px; background: rgba(255,255,255,0.2); border-radius: 50%; display: flex; align-items: center; justify-content: center; .pc-entry-icon { font-size: 20px; } } }
}

.tab-placeholder { height: 100rpx; }
</style>