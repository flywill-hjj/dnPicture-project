<template>
  <scroll-view
    @scrolltolower="handleToLower"
    class="recommend_view"
    scroll-y
    v-if="recommends.length > 0"
  >
    <!-- 推荐 开始 -->
    <view class="recommend_wrap">
      <view class="recommend_item" v-for="item in recommends" :key="item.id">
        <image mode="widthFix" :src="item.thumb"></image>
      </view>
    </view>
    <!-- 推荐 结束 -->

    <!-- 月份 开始 -->
    <view class="months_wrap">
      <view class="months_title">
        <view class="months_title_info">
          <view class="months_info">
            <text> {{ months.DD }} / </text>
            {{ months.MM }} 月
          </view>
          <view class="months_text">{{ months.title }}</view>
        </view>
        <view class="months_title_more">更多 > </view>
      </view>
      <view class="months_content">
        <view class="months_item" v-for="item in months.items" :key="item.id">
          <image
            mode="aspectFill"
            :src="item.thumb + item.rule.replace('$<Height>', 360)"
          ></image>
        </view>
      </view>
    </view>
    <!-- 月份 结束 -->

    <!-- 热门 开始 -->
    <view class="hots_wrap">
      <view class="hots_title">
        <text>热门</text>
      </view>
      <view class="hots_content">
        <view class="hot_item" v-for="item in hots" :key="item.id">
          <image mode="widthFix" :src="item.thumb"></image>
        </view>
      </view>
    </view>
    <!-- 热门 结束 -->
  </scroll-view>
</template>

<script>
import moment from "moment";
export default {
  data() {
    return {
      //推荐列表
      recommends: [],
      //月份
      months: {},
      //热门
      hots: [],
      //请求的参数
      params: {
        //要获取几条数据
        limit: 30,
        //关键字
        order: "hot",
        //要跳过几条
        skip: 0,
      },
      //是否还有下一页
      hasMore: true,
    };
  },
  mounted() {
    this.getList();
  },
  methods: {
    //获取接口的数据
    getList() {
      this.request({
        url: "http://157.122.54.189:9088/image/v3/homepage/vertical",
        data: this.params,
      }).then((result) => {
        //判断还没有有下一页数据
        if (result.res.vertical.length === 0) {
          this.hasMore = false;
          return;
        }

        if (this.recommends.length === 0) {
          //第一次发送的请求
          this.recommends = result.res.homepage[1].items;
          //月份模块
          this.months = result.res.homepage[2];
          //将时间戳 改成 18号/月 moment.js
          this.months.MM = moment(this.months.stime).format("MM");
          this.months.DD = moment(this.months.stime).format("DD");
        }

        //获取热门数据的列表
        //数组拼接 es6
        this.hots = [...this.hots, ...result.res.vertical];
      });
    },
    //滚动条触底事件
    handleToLower() {
      /* 
       1 修改参数 skip+=limit;
       2 重新发送请求 getList()
       3 请求回来成功 hots 数据的叠加
      */
      if (this.hasMore) {
        this.params.skip += this.params.limit;
        this.getList();
      } else {
        //弹窗提示用户
        uni.showToast({
          title: "没有数据了",
          icon: "none",
        });
      }
    },
  },
};
</script>

<style scoped lang="scss">
.recommend_view {
  //height:屏幕的高度 - 头部标题的高度
  height: calc(100vh - 36px);
}

.recommend_wrap {
  display: flex;
  flex-wrap: wrap;
  .recommend_item {
    width: 50%;
    border: 5rpx solid #ffffff;
  }
}
.months_wrap {
  .months_title {
    display: flex;
    justify-content: space-between;
    padding: 20rpx;
    .months_title_info {
      color: $color;
      font-size: 30rpx;
      font-weight: 600;
      display: flex;
      .months_info {
        text {
          font-size: 36rpx;
        }
      }

      .months_text {
        font-size: 34rpx;
        color: #666666;
        margin-left: 30rpx;
      }
    }

    .months_title_more {
      font-size: 24rpx;
      color: $color;
    }
  }

  .months_content {
    display: flex;
    flex-wrap: wrap;
    .months_item {
      width: 33.33%;
      border: 5rpx solid #fff;
    }
  }
}

.hots_wrap {
  .hots_title {
    padding: 20rpx;
    text {
      border-left: 20rpx solid $color;
      padding-left: 20rpx;
      font-size: 34rpx;
      font-weight: 600;
    }
  }

  .hots_content {
    display: flex;
    flex-wrap: wrap;
    .hot_item {
      width: 33.33%;
      border: 5rpx solid #ffffff;
      image {
      }
    }
  }
}
</style>
