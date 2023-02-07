<template>
  <view>
    <view class="search-box">
      <uni-search-bar @input="input" :radius="100" :focus="true" cancelButton="none"></uni-search-bar>
    </view>
    <!-- 搜索建议列表 -->
    <view class="sugg-list" v-if="searchResults.length !== 0">
      <view class="sugg-item" v-for="(item, i) in searchResults" :key="i" @click="gotoDetail(item)">
        <view class="goods-name">{{item.goods_name}}</view>
        <uni-icons type="arrowright" size="16"></uni-icons>
      </view>
    </view>
    
    <!-- 搜索历史 -->
    <view class="history-box" v-else>
      <!-- 标题区域 -->
      <view class="history-title">
        <text>搜索历史</text>
        <uni-icons type="trash" size="16" @click="clean"></uni-icons>
      </view>
      <!-- 列表区域 -->
      <view class="history-list">
        <uni-tag :text="item" v-for="(item, i) in histories" :key="i" customStyle="margin-top:5px; margin-right: 5px;" @click="gotoGoodsList(item)"/>
      </view>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        // 延时器id
        timer: null,
        // 搜索关键词
        kw: '',
        // 搜索建议列表
        searchResults: [],
        // 搜索历史列表
        historyList: []
      }
    },
    onLoad() {
      // 如果kw不存在，就用空数组
      this.historyList = JSON.parse(uni.getStorageSync('kw') || '[]')
    },
    methods: {
      // 输入事件的处理函数
      input(e) {
        // 防抖，当用户500ms不输入内容时，再执行
        clearTimeout(this.timer)
        this.timer = setTimeout(() => {
          this.kw = e
          this.getSearchList()
        }, 500)
      },
      async getSearchList() {
        if (this.kw.length === 0) {
          this.searchResults = []
          return
        }
        
        const {data: res} = await uni.$http.get('/api/public/v1/goods/qsearch', {query: this.kw})
        if (res.meta.status !== 200) return uni.$showMsg()
        this.searchResults = res.message
        
        this.saveSearchHistory()
      },
      gotoDetail(item) {
        uni.navigateTo({
          url: '/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
        })
      },
      saveSearchHistory() {
        // this.historyList.push(this.kw)
        
        const set = new Set(this.historyList)
        set.delete(this.kw)
        set.add(this.kw)
        this.historyList = Array.from(set)
        
        // 持久化历史记录
        uni.setStorageSync('kw', JSON.stringify(this.historyList))
      },
      clean() {
        this.historyList = []
        uni.setStorageSync('kw', '[]')
      },
      gotoGoodsList(kw) {
        uni.navigateTo({
          url: '/subpkg/goods_list/goods_list?query=' + kw
        })
      }
    },
    // 计算属性
    computed: {
      histories() {
        // ... 是为了得到一个副本，防止直接修改原数组
        return [...this.historyList].reverse()
      }
    }
  }
</script>

<style lang="scss">
  .search-box {
    background-color: #C00000;
    position: sticky;
    top: 0;
    z-index: 999;
  }
  
  .sugg-list {
    padding: 0 5px;
    .sugg-item {
      display: flex;
      align-items: center;
      justify-content: space-between;
      font-size: 12px;
      padding: 13px 0;
      border-bottom: 1px solid #efefef;
      
      .goods-name {
        white-space: nowrap;  // 不允许换行
        overflow: hidden;  // 超出的内容隐藏
        text-overflow: ellipsis; // ...
        margin-right: 3px;
      }
    }
  }
  
  .history-box {
    padding: 0 5px;
    .history-title {
      display: flex;
      justify-content: space-between;
      height: 40px;
      align-items: center;
      font-size: 14px;
      border-bottom: 1px solid #efefef;
      margin-bottom: 5px;
    }
    .history-list {
      display: flex;
      flex-wrap: wrap;
      
      .uni-tag {
        
      }
    }
  }
</style>
