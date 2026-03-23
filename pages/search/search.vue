<template>
  <view class="search-page">

    <!-- 状态栏占位（仅App端） -->
    <!-- #ifdef APP-PLUS -->
    <view :style="{ height: statusBarHeight + 'px', background: '#ffffff' }"></view>
    <!-- #endif -->

    <!-- 顶部搜索栏 -->
    <view class="search-header">
      <view class="back-btn" @tap="onBack">
        <text class="back-icon">‹</text>
      </view>
      <view class="search-box">
        <text class="search-icon-text">🔍</text>
        <input
          class="search-input"
          v-model="keyword"
          placeholder="搜索商品"
          placeholder-class="search-placeholder"
          @confirm="onSearch"
          focus
        />
        <view v-if="keyword" class="clear-btn" @tap="clearKeyword">
          <text class="clear-icon">✕</text>
        </view>
      </view>
      <view class="search-submit" @tap="onSearch">
        <text class="search-submit-text">搜索</text>
      </view>
    </view>

    <!-- 筛选栏 -->
    <view class="filter-bar">
      <view
        v-for="filter in filters"
        :key="filter.id"
        class="filter-item"
        :class="{ active: activeFilter === filter.id }"
        @tap="onFilterTap(filter)"
      >
        <text class="filter-text">{{ filter.name }}</text>
        <text v-if="filter.hasArrow" class="filter-arrow">{{ activeFilter === filter.id ? '▴' : '▾' }}</text>
      </view>
      <view class="filter-right" @tap="toggleViewMode">
        <text class="filter-view-icon">{{ isGridView ? '☰' : '⊞' }}</text>
      </view>
    </view>

    <!-- 搜索结果统计 -->
    <view class="result-stats">
      <text class="result-text">共找到 <text class="result-count">{{ resultList.length }}</text> 件 "{{ keyword }}" 相关商品</text>
    </view>

    <!-- 商品列表 -->
    <scroll-view
      scroll-y
      class="goods-scroll"
      @scrolltolower="onLoadMore"
      :show-scrollbar="false"
    >
      <!-- 网格视图 -->
      <view v-if="isGridView" class="goods-grid">
        <view
          v-for="goods in resultList"
          :key="goods.id"
          class="grid-item"
          @tap="onGoodsTap(goods)"
        >
          <view class="grid-img-wrap" :style="{ background: goods.color }">
            <text class="grid-mock-icon">{{ goods.icon }}</text>
            <view v-if="goods.tag" class="grid-tag">
              <text class="grid-tag-text">{{ goods.tag }}</text>
            </view>
            <!-- 收藏按钮 -->
            <view class="fav-btn" @tap.stop="onFavTap(goods)">
              <text class="fav-icon">{{ goods.fav ? '❤' : '♡' }}</text>
            </view>
          </view>
          <view class="grid-info">
            <view class="shop-name-row">
              <text class="shop-name">{{ goods.shopName }}</text>
            </view>
            <text class="grid-name">{{ goods.name }}</text>
            <view class="grid-price-row">
              <view class="price-left">
                <text class="price-symbol">¥</text>
                <text class="grid-price">{{ goods.price }}</text>
              </view>
              <text class="grid-sold">{{ goods.sold }}人付款</text>
            </view>
            <view class="grid-tags">
              <text v-for="t in goods.badges" :key="t" class="grid-badge">{{ t }}</text>
            </view>
          </view>
        </view>
      </view>

      <!-- 列表视图 -->
      <view v-else class="goods-list">
        <view
          v-for="goods in resultList"
          :key="goods.id"
          class="list-item"
          @tap="onGoodsTap(goods)"
        >
          <view class="list-img-wrap" :style="{ background: goods.color }">
            <text class="list-mock-icon">{{ goods.icon }}</text>
            <view v-if="goods.tag" class="list-tag">
              <text class="list-tag-text">{{ goods.tag }}</text>
            </view>
          </view>
          <view class="list-info">
            <text class="list-name">{{ goods.name }}</text>
            <view class="list-badges">
              <text v-for="t in goods.badges" :key="t" class="list-badge">{{ t }}</text>
            </view>
            <text class="list-shop">{{ goods.shopName }}</text>
            <view class="list-bottom">
              <view class="list-price-wrap">
                <text class="list-price-symbol">¥</text>
                <text class="list-price">{{ goods.price }}</text>
                <text class="list-origin">¥{{ goods.originPrice }}</text>
              </view>
              <view class="list-actions">
                <view class="fav-btn-list" @tap.stop="onFavTap(goods)">
                  <text class="fav-icon-list">{{ goods.fav ? '❤' : '♡' }}</text>
                </view>
                <view class="cart-btn" @tap.stop="onAddCart(goods)">
                  <text class="cart-btn-text">加购</text>
                </view>
              </view>
            </view>
            <text class="list-sold">已售 {{ goods.sold }}</text>
          </view>
        </view>
      </view>

      <!-- 加载更多 -->
      <view class="load-more">
        <text v-if="isLoading" class="load-more-text">加载中...</text>
        <text v-else-if="noMore" class="load-more-text">— 没有更多了 —</text>
        <text v-else class="load-more-hint">上拉加载更多</text>
      </view>

    </scroll-view>

  </view>
</template>

<script>
// mock数据生成函数
function generateMockGoods(keyword, count = 20) {
  const icons = ['👟', '🎧', '☕', '🥤', '👕', '⌚', '💼', '📱', '🖥️', '🎮', '👜', '🏃', '🧴', '🌂', '📚']
  const colors = ['#E3F2FD', '#F3E5F5', '#E8F5E9', '#FFF3E0', '#FCE4EC', '#E0F7FA', '#FFF8E1', '#F9FBE7', '#EFEBE9']
  const shops = ['优品旗舰店', '好货专营店', '品质优选', '厂家直销', '正品保证店', '全球购', '品牌直营']
  const tags = ['券后价', '爆款', '新品', '热销', null, null]
  const badges = [['包邮', '7天退换'], ['假一赔三'], ['包邮'], ['7天退换', '正品'], []]

  return Array.from({ length: count }, (_, i) => ({
    id: i + 1,
    name: `${keyword}相关商品 - 高品质${['男女款', '通用款', '旗舰版', '经典款', '限量版'][i % 5]}优质好物精选`,
    price: (Math.random() * 500 + 20).toFixed(0),
    originPrice: (Math.random() * 800 + 200).toFixed(0),
    sold: `${(Math.random() * 10 + 0.1).toFixed(1)}万`,
    icon: icons[i % icons.length],
    color: colors[i % colors.length],
    shopName: shops[i % shops.length],
    tag: tags[i % tags.length],
    badges: badges[i % badges.length],
    fav: false,
  }))
}

export default {
  name: 'SearchPage',
  data() {
    return {
      statusBarHeight: 0,
      keyword: '',
      isGridView: true,
      activeFilter: 1,
      isLoading: false,
      noMore: false,
      page: 1,

      filters: [
        { id: 1, name: '综合', hasArrow: false },
        { id: 2, name: '销量', hasArrow: false },
        { id: 3, name: '价格', hasArrow: true },
        { id: 4, name: '筛选', hasArrow: true },
      ],

      resultList: [],
    }
  },

  onLoad(options) {
    // #ifdef APP-PLUS
    const info = uni.getSystemInfoSync()
    this.statusBarHeight = info.statusBarHeight
    // #endif

    if (options.keyword) {
      this.keyword = decodeURIComponent(options.keyword)
    }
    this.loadData()
  },

  methods: {
    loadData() {
      this.resultList = generateMockGoods(this.keyword, 20)
    },

    onBack() {
      uni.navigateBack()
    },

    onSearch() {
      if (!this.keyword.trim()) {
        uni.showToast({ title: '请输入搜索关键词', icon: 'none' })
        return
      }
      this.page = 1
      this.noMore = false
      this.loadData()
    },

    clearKeyword() {
      this.keyword = ''
    },

    onFilterTap(filter) {
      this.activeFilter = filter.id
    },

    toggleViewMode() {
      this.isGridView = !this.isGridView
    },

    onGoodsTap(goods) {
      uni.showToast({ title: '跳转商品详情', icon: 'none' })
    },

    onFavTap(goods) {
      goods.fav = !goods.fav
      uni.showToast({
        title: goods.fav ? '已收藏' : '已取消收藏',
        icon: 'none',
        duration: 1000
      })
    },

    onAddCart(goods) {
      uni.showToast({ title: '已加入购物车', icon: 'success' })
    },

    onLoadMore() {
      if (this.isLoading || this.noMore) return
      this.isLoading = true
      setTimeout(() => {
        if (this.page >= 3) {
          this.noMore = true
        } else {
          const more = generateMockGoods(this.keyword, 10)
          this.resultList = [...this.resultList, ...more]
          this.page++
        }
        this.isLoading = false
      }, 1000)
    },
  }
}
</script>

<style lang="scss">
.search-page {
  min-height: 100vh;
  background: #F5F5F5;
  display: flex;
  flex-direction: column;
}

/* ==================== 搜索头部 ==================== */
.search-header {
  display: flex;
  align-items: center;
  background: #ffffff;
  padding: 16rpx 20rpx;
  gap: 16rpx;

  .back-btn {
    width: 52rpx;
    height: 52rpx;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;

    .back-icon {
      font-size: 52rpx;
      color: #333333;
      line-height: 1;
    }
  }

  .search-box {
    flex: 1;
    display: flex;
    align-items: center;
    background: #F5F5F5;
    border-radius: 50rpx;
    padding: 0 20rpx;
    height: 68rpx;
    gap: 12rpx;

    .search-icon-text {
      font-size: 28rpx;
      flex-shrink: 0;
    }

    .search-input {
      flex: 1;
      font-size: 26rpx;
      color: #333333;
      height: 100%;
    }

    .search-placeholder {
      color: #BBBBBB;
      font-size: 26rpx;
    }

    .clear-btn {
      width: 36rpx;
      height: 36rpx;
      background: #CCCCCC;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      flex-shrink: 0;

      .clear-icon {
        font-size: 20rpx;
        color: #ffffff;
      }
    }
  }

  .search-submit {
    flex-shrink: 0;

    .search-submit-text {
      font-size: 28rpx;
      color: #FF5000;
      font-weight: bold;
    }
  }
}

/* ==================== 筛选栏 ==================== */
.filter-bar {
  display: flex;
  align-items: center;
  background: #ffffff;
  border-top: 1rpx solid #F0F0F0;
  border-bottom: 1rpx solid #F0F0F0;
  margin-top: 2rpx;

  .filter-item {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    height: 80rpx;
    gap: 6rpx;
    border-bottom: 4rpx solid transparent;

    &.active {
      border-bottom-color: #FF5000;

      .filter-text {
        color: #FF5000;
        font-weight: bold;
      }

      .filter-arrow {
        color: #FF5000;
      }
    }

    .filter-text {
      font-size: 26rpx;
      color: #333333;
    }

    .filter-arrow {
      font-size: 18rpx;
      color: #999999;
    }
  }

  .filter-right {
    width: 80rpx;
    display: flex;
    align-items: center;
    justify-content: center;
    height: 80rpx;
    border-left: 1rpx solid #F0F0F0;

    .filter-view-icon {
      font-size: 32rpx;
      color: #666666;
    }
  }
}

/* ==================== 结果统计 ==================== */
.result-stats {
  padding: 16rpx 24rpx;
  background: #F5F5F5;

  .result-text {
    font-size: 22rpx;
    color: #999999;

    .result-count {
      color: #FF5000;
      font-weight: bold;
    }
  }
}

/* ==================== 滚动容器 ==================== */
.goods-scroll {
  flex: 1;
  height: 0;
  flex-grow: 1;
}

/* ==================== 网格视图 ==================== */
.goods-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 16rpx;
  padding: 8rpx 16rpx 16rpx;

  .grid-item {
    width: calc(50% - 8rpx);
    background: #ffffff;
    border-radius: 12rpx;
    overflow: hidden;
    box-shadow: 0 2rpx 8rpx rgba(0,0,0,0.06);

    .grid-img-wrap {
      width: 100%;
      height: 220rpx;
      display: flex;
      align-items: center;
      justify-content: center;
      position: relative;

      .grid-mock-icon {
        font-size: 90rpx;
      }

      .grid-tag {
        position: absolute;
        top: 12rpx;
        left: 12rpx;
        background: #FF5000;
        border-radius: 6rpx;
        padding: 4rpx 10rpx;

        .grid-tag-text {
          font-size: 18rpx;
          color: #ffffff;
        }
      }

      .fav-btn {
        position: absolute;
        top: 12rpx;
        right: 12rpx;
        width: 48rpx;
        height: 48rpx;
        background: rgba(255,255,255,0.9);
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;

        .fav-icon {
          font-size: 26rpx;
          color: #FF5000;
        }
      }
    }

    .grid-info {
      padding: 14rpx 16rpx;

      .shop-name-row {
        margin-bottom: 6rpx;

        .shop-name {
          font-size: 18rpx;
          color: #999999;
          background: #F5F5F5;
          padding: 2rpx 8rpx;
          border-radius: 4rpx;
        }
      }

      .grid-name {
        font-size: 24rpx;
        color: #333333;
        line-height: 1.4;
        display: -webkit-box;
        -webkit-box-orient: vertical;
        -webkit-line-clamp: 2;
        overflow: hidden;
      }

      .grid-price-row {
        display: flex;
        align-items: center;
        justify-content: space-between;
        margin-top: 10rpx;

        .price-left {
          display: flex;
          align-items: baseline;

          .price-symbol {
            font-size: 20rpx;
            color: #FF5000;
            font-weight: bold;
          }

          .grid-price {
            font-size: 34rpx;
            color: #FF5000;
            font-weight: bold;
          }
        }

        .grid-sold {
          font-size: 18rpx;
          color: #999999;
        }
      }

      .grid-tags {
        display: flex;
        flex-wrap: wrap;
        gap: 8rpx;
        margin-top: 10rpx;

        .grid-badge {
          font-size: 18rpx;
          color: #FF5000;
          border: 1rpx solid #FF5000;
          padding: 2rpx 8rpx;
          border-radius: 4rpx;
        }
      }
    }
  }
}

/* ==================== 列表视图 ==================== */
.goods-list {
  padding: 8rpx 0;

  .list-item {
    display: flex;
    background: #ffffff;
    margin-bottom: 16rpx;
    padding: 24rpx;
    gap: 20rpx;

    .list-img-wrap {
      width: 200rpx;
      height: 200rpx;
      border-radius: 12rpx;
      flex-shrink: 0;
      display: flex;
      align-items: center;
      justify-content: center;
      position: relative;

      .list-mock-icon {
        font-size: 80rpx;
      }

      .list-tag {
        position: absolute;
        top: 8rpx;
        left: 8rpx;
        background: #FF5000;
        border-radius: 6rpx;
        padding: 2rpx 8rpx;

        .list-tag-text {
          font-size: 18rpx;
          color: #ffffff;
        }
      }
    }

    .list-info {
      flex: 1;
      display: flex;
      flex-direction: column;

      .list-name {
        font-size: 26rpx;
        color: #333333;
        line-height: 1.5;
        display: -webkit-box;
        -webkit-box-orient: vertical;
        -webkit-line-clamp: 2;
        overflow: hidden;
      }

      .list-badges {
        display: flex;
        flex-wrap: wrap;
        gap: 8rpx;
        margin-top: 10rpx;

        .list-badge {
          font-size: 18rpx;
          color: #FF5000;
          border: 1rpx solid #FF5000;
          padding: 2rpx 8rpx;
          border-radius: 4rpx;
        }
      }

      .list-shop {
        font-size: 20rpx;
        color: #999999;
        margin-top: 8rpx;
      }

      .list-bottom {
        display: flex;
        align-items: center;
        justify-content: space-between;
        margin-top: auto;
        padding-top: 16rpx;

        .list-price-wrap {
          display: flex;
          align-items: baseline;
          gap: 4rpx;

          .list-price-symbol {
            font-size: 20rpx;
            color: #FF5000;
            font-weight: bold;
          }

          .list-price {
            font-size: 36rpx;
            color: #FF5000;
            font-weight: bold;
          }

          .list-origin {
            font-size: 20rpx;
            color: #BBBBBB;
            text-decoration: line-through;
          }
        }

        .list-actions {
          display: flex;
          align-items: center;
          gap: 16rpx;

          .fav-btn-list {
            width: 52rpx;
            height: 52rpx;
            display: flex;
            align-items: center;
            justify-content: center;

            .fav-icon-list {
              font-size: 30rpx;
              color: #FF5000;
            }
          }

          .cart-btn {
            background: #FF5000;
            border-radius: 30rpx;
            padding: 10rpx 24rpx;

            .cart-btn-text {
              font-size: 22rpx;
              color: #ffffff;
            }
          }
        }
      }

      .list-sold {
        font-size: 20rpx;
        color: #999999;
        margin-top: 6rpx;
      }
    }
  }
}

/* ==================== 加载更多 ==================== */
.load-more {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 40rpx 0 60rpx;

  .load-more-text, .load-more-hint {
    font-size: 24rpx;
    color: #BBBBBB;
  }
}
</style>