<template>
    <view>
        <myNewsList
            @change="getNewsList"
            :maxIndex="maxIndex"
            :news="news"
            ref="newsListRef"
        ></myNewsList>

        <view v-if="flag == 'history'" class="btn" @click="clear"
            >清空历史浏览</view
        >
    </view>
</template>

<script>
import { unify } from "@/service/api/unify.js";
export default {
    data() {
        return {
            news: [],
            flag: null,
            title: "",
            maxIndex: null,
            pageSize: 10,
            pageIndex: 0,
            text: null,
            k: null,
        };
    },
    async onLoad(config) {
        this.flag = config.title; // 设置状态变量
        let countres; // 数据长度

        if (this.flag == "likes") {
            // 收藏页
            this.title = "我的收藏";
            countres = await unify({
                name: "likes",
                conut: true,
                strict: true,
                data: {
                    uid: uni.getStorageSync("uid"),
                },
            });
        } else if (this.flag == "search") {
            // 搜索页
            this.title = "新闻搜索";
            this.k = config.flag;
            this.text = config.text; // 搜索内容
            let data = {
                name: "news",
                conut: true,
                data: {},
            };
            data.data[config.flag] = config.text;
            countres = await unify(data);
        } else {
            // 历史新闻数据
            this.title = "历史浏览";
            let news = uni.getStorageSync("news") || [];
            this.news = news;
        }

        if (this.title != "历史浏览") {
            this.maxIndex = Math.floor(countres.data[0].rows / 8 + 1);
            this.getNewsList();
        }
         console.log("this.maxIndex", this.maxIndex);
        uni.setNavigationBarTitle({
            title: this.title,
        });
    },
    onReachBottom() {
        if (this.pageIndex < this.maxIndex && this.title != "历史浏览") {
            this.pageIndex++;
            this.getNewsList();
        }
        if (this.title == "历史浏览") {
            this.$refs.newsListRef.loading = 2;
        }
    },
    methods: {
        clear() {
            uni.setStorageSync("news", []);
            this.news = [];
        },
        async getNewsList() {
            let res;
            if (this.flag == "likes") {
                // 收藏新闻数据
                let uid = uni.getStorageSync("uid");
                res = await unify({
                    name: "likes",
                    strict: true,
                    pageSize: this.pageSize,
                    pageIndex: this.pageIndex,
                    data: {
                        "likes.uid": uid,
                    },
                });
                //
            } else if (this.flag == "search") {
                // 搜索新闻数据
                let data = {
                    name: "news",
                    pageSize: this.pageSize,
                    pageIndex: this.pageIndex,
                    data: {},
                };
                data.data[this.k] = this.text;
                res = await unify(data);
            }
            if (res.code == 400) {
                this.$refs.newsListRef.loading = 2;
                return;
            }
            if (this.title != "历史浏览")
                this.news = [...this.news, ...res.data];
        },
    },
};
</script>

<style lang="less">
.info {
    text-align: center;
    width: 100%;
    height: 40px;
    line-height: 40px;
}

.news {
    padding-bottom: 44px;
}

.btn {
    background-color: #efefef;
    position: fixed;
    bottom: 0;
    left: 0;
    text-align: center;
    width: 100%;
    line-height: 40px;
    height: 40px;
}
</style>
