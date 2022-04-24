<template>
  <view class="index">
    <AtNoticebar>
      通告栏：0.0.1版本预览上线，后续功能持续规划中。
    </AtNoticebar>
    <view
      v-for="item in articles"
      v-bind:key="item.req_id"
      class="articleContainer"
      @tap="createPoster(item)"
    >
      <view class="articleTitle">{{ item.article_info.title }}</view>
      <view class="articleCoverAndDescribe">
        <view class="articleDescribe">
          {{ item.article_info.brief_content }}
        </view>
        <image class="articleCover" :src="item.article_info.cover_image" />
      </view>
      <view class="articleFoot">
        <view style="display: flex; align-items: center">
          <view class="at-icon at-icon-eye"> </view>
          <view style="margin-left: 10rpx">{{
            item.article_info.view_count
          }}</view>
          <view class="at-icon at-icon-star-2" style="margin-left: 15rpx">
          </view>
          <view style="margin-left: 10rpx">{{
            item.article_info.digg_count
          }}</view>
          <view class="at-icon at-icon-message" style="margin-left: 15rpx">
          </view>
          <view style="margin-left: 10rpx">{{
            item.article_info.comment_count
          }}</view>
        </view>
        <view>
          <AtTag
            v-for="tag in item.tags"
            v-bind:key="tag.id"
            type="primary"
            style="margin-left: 10rpx"
            >{{ tag.tag_name }}</AtTag
          >
        </view>
      </view>
    </view>
    <AtLoadMore @click="handleLoadMore" :status="status" />
    <AtCurtain :isOpened="posterPath !== ''" @close="posterPath = ''">
      <image :src="posterPath" mode="widthFix" @tap="savePoster" />
    </AtCurtain>

    <PosterBuilder
      v-if="startDraw"
      :config="posterConfig"
      @success="drawSuccess"
      @fail="drawFail"
    />
  </view>
</template>

<script>
import { AtLoadMore } from "taro-ui-vue3";
import { AtNoticebar } from "taro-ui-vue3";
import { AtTag } from "taro-ui-vue3";
import { AtCurtain } from "taro-ui-vue3";
import { AtIcon } from 'taro-ui-vue3'
import PosterBuilder from "../../components/PosterBuilder/index.vue";

import Taro from "@tarojs/taro";
import "./index.scss";
import { toRaw } from "vue-demi";

export default {
  name: "Index",
  components: {
    AtLoadMore,
    AtNoticebar,
    AtTag,
    AtCurtain,
    PosterBuilder,
  },
  data() {
    return {
      status: "more",
      articles: [],
      page: 0,
      startDraw: false,
      posterPath: "",
      posterConfig: {
        width: 750,
        height: 975,
        backgroundColor: "#4c1e1a",
        debug: false,
        blocks: [
          {
            x: 32,
            y: 40,
            width: 686,
            height: 160,
            paddingLeft: 0,
            paddingRight: 0,
            backgroundColor: "#FFFFFF",
            borderRadius: 32,
            zIndex: 10,
          },
          {
            x: 32,
            y: 620,
            width: 686,
            height: 302,
            paddingLeft: 0,
            paddingRight: 0,
            borderRadiusGroup: [0, 0, 16, 16],
            backgroundColor: "#FFFFFF",
            zIndex: 11,
          },
        ],
        texts: [],
        images: [],
      },
    };
  },
  async mounted() {
    await this.getArticleApi();
  },
  methods: {
    async getArticleApi() {
      const {
        statusCode,
        data: { data },
      } = await Taro.request({
        url: "https://api.juejin.cn/content_api/v1/article/query_list",
        method: "POST",
        data: {
          user_id: "3966693685871694",
          sort_type: 2,
          cursor: `${this.page * 10}`,
        },
      });
      if (statusCode === 200) {
        if (data && data.length > 0) {
          if (this.page === 0) this.articles = [];
          data.forEach((item) => {
            this.articles.push(item);
          });
          this.status = "more";
        } else {
          this.status = "noMore";
        }
      }
    },
    async handleLoadMore() {
      this.status = "loading";
      this.page++;
      await this.getArticleApi();
    },
    createPoster(article) {
      this.startDraw = true;
      Taro.showLoading({ title: "正在生成海报" });
      this.posterConfig.texts = [];
      this.posterConfig.texts.push({
        x: 216,
        y: 75,
        text: "小鑫同学",
        width: 380,
        lineNum: 2, // 最多几行
        fontSize: 36,
        fontWeight: "bold",
        color: "#1A171B",
        zIndex: 11,
      });
      this.posterConfig.texts.push({
        x: 216,
        y: 134,
        text: "最新推文来YE！",
        width: 380,
        fontSize: 28,
        color: "#7C7D7A",
        zIndex: 11,
      });
      this.posterConfig.texts.push({
        x: 64,
        y: 650,
        text: `${article.article_info.title}`,
        fontSize: 38,
        lineHeight: 45,
        color: "#ED2D2B",
        lineNum: 2,
        width: 586,
        fontWeight: "bold",
        zIndex: 12,
      });
      this.posterConfig.texts.push({
        x: 64,
        y: 780,
        text: `${article.article_info.brief_content}`,
        fontSize: 34,
        width: 586,
        lineHeight: 40,
        color: "#282925",
        lineNum: 3,
        zIndex: 12,
      });
      this.posterConfig.images = [];
      this.posterConfig.images.push({
        x: 64,
        y: 60,
        width: 120,
        height: 120,
        borderRadius: 60,
        url: `${article.author_user_info.avatar_large}`,
        zIndex: 11,
      });
      this.posterConfig.images.push({
        x: 32,
        y: 242,
        width: 686,
        height: 346,
        url: `${article.article_info.cover_image}`,
        borderRadiusGroup: [16, 16, 0, 0],
        zIndex: 11,
      });
    },
    drawSuccess(result) {
      this.startDraw = false;
      console.log("绘制好了", result);
      const { tempFilePath, errMsg } = result;
      Taro.hideLoading();
      if (errMsg === "canvasToTempFilePath:ok") {
        this.posterPath = tempFilePath;
      } else {
        Taro.showToast({
          title: "失败，请稍后重试",
          icon: "none",
          duration: 2500,
        });
      }
    },
    drawFail(result) {
      this.startDraw = false;
      console.log("绘制失败", result);
      Taro.hideLoading();
    },
    savePoster() {
      Taro.saveImageToPhotosAlbum({
        filePath: this.posterPath,
        success() {
          Taro.showToast({
            title: "已保存到相册",
            icon: "success",
            duration: 2000,
          });
        },
      });
    },
  },
};
</script>
