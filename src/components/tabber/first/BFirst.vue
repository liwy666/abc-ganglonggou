<template>
	<div class="main">
		<!--搜索框-->
		<mySearch></mySearch>
		<!--头部轮播图-->
		<div class="my-van-swipe-box">
			<van-swipe :autoplay="3000" indicator-color="white">
				<van-swipe-item v-for="(item, index) in index_ad_list.swipe_img" :key="index">
					<img :src="item.ad_img" @click="toControl(item)"/>
				</van-swipe-item>
			</van-swipe>
		</div>
		<!--公告-->
		<div class="notice-box">
			<van-notice-bar
				:text="index_ad_list.notice_text.text"
				:scrollable="true"
				color="rgb(0,0,0)"
				background="rgb(255,255,255)"
				:left-icon="index_ad_list.notice_text.ad_img"
			/>
		</div>
		<!--分类区域-->
		<div class="classify-box">
			<div class="classify" v-for="(item) in index_ad_list.classify" :key="item.id">
				<img v-lazy="item.ad_img" alt="" @click="toControl(item)">
			</div>
		</div>
		<!--孤立-->
		<div class="banner-box">
			<img v-lazy="index_ad_list.banner.ad_img" alt="" @click="toControl(index_ad_list.banner)">
		</div>
		<!--合作伙伴-->
		<div class="align-box">
			<div class="align" v-for="(item) in index_ad_list.align" :key="item.id">
				<img v-lazy="item.ad_img" alt="" @click="toControl(item)">
			</div>
		</div>
		<!--服务说明-->
		<div class="banner-box">
			<img v-lazy="index_ad_list.server.ad_img" alt="" @click="toControl(index_ad_list.server)">
		</div>
		<!--每周特惠-->
		<div class="weekly-box">
			<div class="banner">
				<img v-lazy="index_ad_list.weekly_banner.ad_img" alt="">
			</div>
			<div class="base">
				<div class="goods-cart" v-for="(item) in index_ad_list.weekly_goods" :key="item.id" @click="toControl(item)">
					<div class="goods-img"><img v-lazy="item.ad_img" alt="" >
						<div class="hot-img"><img src="../../../assets/hot.png" alt=""></div>
					</div>
					<div class="goods-name">{{item.goods_name}}</div>
					<div class="goods-price-box">
						<div class="l">
							<p class="origin-goods-price">￥{{item.origin_goods_price}}</p>
							<p class="goods-price">￥{{item.goods_price}}</p>
						</div>
						<div class="r">
							<img src="../../../assets/rob.png" alt="">
						</div>
					</div>
				</div>
			</div>
		</div>
		<!--专区-->
		<div class="special-area-box" v-for="(item,i) in special_area_list" :key="i">
			<div class="banner"><img v-lazy="item.short_banner.ad_img" alt=""></div>
			<div class="banner"><img v-lazy="item.tall_banner.ad_img" alt="" @click="toControl(item.tall_banner)"></div>
			<div class="base">
				<div class="goods-cart" v-for="(item2) in item.goods_list" :key="item2.id"  @click="toControl(item2)">
					<div class="goods-img">
						<div class="hot-img"><img src="../../../assets/rob_ing.png" alt=""></div>
						<img v-lazy="item2.ad_img" alt="">
					</div>
					<div class="goods-name">{{item2.goods_name}}</div>
				</div>
			</div>
		</div>
		<!--压屏广告-->
		<md-landscape v-model="show_pic">
			<img :src="index_ad_list.pop_img_url">
		</md-landscape>
		<div class="d"></div>
	</div>
</template>
<script>
    import mySearch from "./sub/my-sub-search.vue";//搜索组件
    import oneGoods from "../../sub/my-one-goods";//搜索组件
    export default {
        data() {
            return {
                get_info: {},
                coupon_swiper: {
                    slidesPerView: 2.5,
                    spaceBetween: 10,
                },
                radio: "1",
                cat_index: 0,
                show_pic: false,
            };
        },
        created() {
            this.getIndexAd();
        },
        activated() {
            /*获取Url参数信息*/
            let goods_id = this.GetQueryString('gl_goods_id');
            if (goods_id != null && goods_id.length > 1) {
                if ((/(^[1-9]\d*$)/.test(parseInt(goods_id)))) {//验证正整数
                    this.$router.push('/goods/' + goods_id);
                }
            }
        },
        computed: {
            index_ad_list: {
                get: function () {
                    let result = {
                        swipe_img: [],
                        notice_text: {},
                        pop_img_url: '',
                        classify: [],
                        banner: {},
                        align: [],
                        server: {},
                        weekly_banner: {},
                        weekly_goods: [],
                    };
                    if (JSON.stringify(this.get_info) !== '{}') {
                        this.get_info.ad_list.forEach(item => {
                            if (item.position_type === '顶部轮播') {
                                result.swipe_img.push(item);
                            } else if (item.position_type === '压屏广告') {
                                result.pop_img_url = item.ad_img;
                            } else if (item.position_type === '公告') {
                                result.notice_text = item;
                            } else if (item.position_type === '分类区域') {
                                result.classify.push(item);
                            } else if (item.position_type === '孤立通栏') {
                                result.banner = item;
                            } else if (item.position_type === '合作伙伴') {
                                result.align.push(item);
                            } else if (item.position_type === '服务说明') {
                                result.server = item;
                            } else if (item.position_type === '每周特惠横图') {
                                result.weekly_banner = item;
                            } else if (item.position_type === '每周特惠商品') {
                                result.weekly_goods.push(item);
                            }
                        });
                    }
                    return result;
                }
            },
            special_area_list: {
                get: function () {
                    let newArr = [],
                        types = {},
                        i, j, cur;
                    if (JSON.stringify(this.get_info) !== '{}') {
                        for (i = 0, j = this.get_info.ad_list.length; i < j; i++) {
                            cur = this.get_info.ad_list[i];
                            if (!(cur.position_type_name in types)) {
                                if (cur.position_type_name !== "" && cur.position_type_name !== null) {
                                    types[cur.position_type_name] = {
                                        position_type_name: cur.position_type_name,
                                        short_banner: {},
                                        tall_banner: {},
                                        goods_list: [],
                                    };
                                    newArr.push(types[cur.position_type_name]);
                                }

                            }
                            if (cur.position_type_name !== "" && cur.position_type_name !== null) {
                                switch (cur.position_type) {
                                    case '专区短横图':
                                        types[cur.position_type_name].short_banner = cur;
                                        break;
                                    case '专区长横图':
                                        types[cur.position_type_name].tall_banner = cur;
                                        break;
                                    case '专区商品':
                                        types[cur.position_type_name].goods_list.push(cur);
                                        break;
                                }
                            }


                        }
                    }
                    return newArr;
                }
            }
        },
        components: {
            mySearch,//搜索组件
            oneGoods,//单个商品
        },
        methods: {
            getIndexAd() {
                this.$fetch("get_index_info", {into_type: this.$store.getters.getIntoType}).then((msg) => {
                    if (msg) {
                        this.get_info = msg;
                        this.$set(this.$store.state, 'goods_list', msg.goods_list);//赋值商品列表
                        this.showPic();
                    }

                })
            },

            showPic() {
                let time = localStorage.getItem('glShowPicTime') || 0;
                if (time < parseInt(new Date().getTime())) {
                    localStorage.setItem('glShowPicTime', parseInt(new Date().getTime()) + 86400000);
                    this.show_pic = true;
                }
            },
            toControl(ad_info) {
                if (ad_info.ad_type === "商品ID") {
                    if (ad_info.goods_id != null && ad_info.goods_id !== '' && ad_info.goods_id !== 0) {
                        this.$router.push('goods/' + ad_info.goods_id)
                    }
                } else if (ad_info.ad_type === "分类ID") {
                    if (ad_info.cat_id != null && ad_info.cat_id !== '' && ad_info.cat_id !== 0) {
                        this.$router.push({
                            path: 'goodsList',
                            query: {type: 'cat', cat_id: ad_info.cat_id, keyword: "", back_number: -1}
                        })
                    }
                } else if (ad_info.ad_type === "搜索关键词") {
                    if (ad_info.text != null && ad_info.text !== '') {
                        this.$router.push({
                            path: 'goodsList',
                            query: {type: 'search', cat_id: -1, keyword: ad_info.text, back_number: -1}
                        })
                    }
                } else if (ad_info.ad_type === "优惠券ID") {
                    if (ad_info.index_coupon_id != null && ad_info.index_coupon_id !== '' && ad_info.index_coupon_id !== 0) {
                        let toast1 = this.$toast.loading({
                            mask: true,
                            message: '领取中...',
                            duration: 0
                        });
                        this.$post('user_get_coupon', {
                            coupon_id: ad_info.index_coupon_id,
                            user_token: this.$store.getters.getUserToken
                        })
                            .then((msg) => {
                                toast1.clear();
                                this.$toast(msg);
                            })
                    }
                } else if (ad_info.ad_type === "外链接") {
                    if (ad_info.text != null && ad_info.text !== '') {
                        this.$router.push({
                            path: 'myIframe',
                            query: {src: ad_info.index_url}
                        })
                    }
                } else if (ad_info.ad_type === "内部文章") {
                    console.log(ad_info);
                    if (ad_info.index_article_id != null && ad_info.index_article_id !== '' && ad_info.index_article_id !== 0) {
                        this.$router.push('article/' + ad_info.index_article_id)
                    }
                } else {
                    console.log(ad_info.ad_type);
                    return false;
                }
            },
            toTestControl() {
                return false;
            },
            GetQueryString(name) {
                let reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)");
                let r = window.location.search.substr(1).match(reg);
                if (r != null) return unescape(r[2]);
                return null;
            }
        },
    };
</script>
<style lang="scss" scoped>
	.main {
		background-color: rgb(241, 246, 249);
		/*顶部轮播*/
		.my-van-swipe-box {
			height: 180px;
			overflow: hidden;

			.van-swipe {
				img {
					width: 100%;
				}
			}
		}

		.banner-box {
			width: 100%;

			img {
				width: 100%;
			}
		}

		.classify-box {
			display: flex;
			justify-content: flex-start;
			flex-wrap: wrap;

			.classify {
				width: 33%;

				img {
					width: 100%;
					height: 100%;
				}
			}
		}

		.align-box {
			display: flex;
			justify-content: flex-start;
			flex-wrap: wrap;
			background-color: rgb(243, 248, 241);

			.align {
				width: 20%;

				img {
					width: 100%;
				}
			}
		}

		.weekly-box {
			.banner {
				width: 100%;

				img {
					width: 100%;
				}
			}

			.base {
				width: 100%;
				display: flex;
				justify-content: flex-start;
				flex-wrap: wrap;

				.goods-cart {
					width: 25%;
					background-color: white;
					.goods-img {
						width: 100%;
						position: relative;
						img {
							width: 100%;
						}

						.hot-img {
							height: 30px;
							width: 30px;
							position: absolute;
							bottom: 10px;
							right: 0;
							img {
								width: 100%;
							}
						}
					}
					.goods-name{
						border: 0px solid black;
						position: relative;
						box-sizing: border-box;
						display: -webkit-box;
						-webkit-box-orient: vertical;
						flex-direction: column;
						align-content: flex-start;
						flex-shrink: 0;
						font-size: 11px;
						text-align: left;
						line-height: 17px;
						height: 36px;
						color: rgb(62, 62, 62);
						-webkit-line-clamp: 2;
						overflow: hidden;
					}
					.goods-price-box{
						display: flex;
						align-items: center;
						justify-content: space-around;
						.l{
							.goods-price{
								color: #007aff;
								font-size: 12px;
								padding-left: 5px;
								height: 20px;
								line-height: 20px;
								font-weight: bold;
							}
							.origin-goods-price{
								color: #c8c9cc;
								text-decoration:line-through;
								font-size: 10px;
								padding-left: 5px;
							}
						}
						.r{
							width: 20px;
							height: 20px;
							img{
								width: 100%;
							}
						}
					}
				}
			}
		}

		.special-area-box {
			.banner {
				width: 100%;

				img {
					width: 100%;
				}
			}

			.base {
				width: 100%;
				display: flex;
				justify-content: flex-start;
				flex-wrap: wrap;
				.goods-cart {
					width: 25%;
					background-color: white;
					.goods-img{
						position: relative;
						width: 100%;
						img {
							width: 100%;
						}
						.hot-img{
							position: absolute;
							width: 35px;
						}
					}
					.goods-name{
						padding: 5px;
						margin-bottom: 5px;
						border: 0px solid black;
						position: relative;
						box-sizing: border-box;
						display: -webkit-box;
						-webkit-box-orient: vertical;
						flex-direction: column;
						align-content: flex-start;
						flex-shrink: 0;
						font-size: 10px;
						text-align: left;
						line-height: 15px;
						height: 36px;
						color: rgb(62, 62, 62);
						-webkit-line-clamp: 2;
						overflow: hidden;
					}
				}
			}
		}
		.d{
			width: 100%;
			height: 80px;
		}
	}
</style>
<style lang="scss">
	.md-landscape-content {
		width: 375px;
		text-align: center;

		img {
			width: 80%;
		}
	}

	.main {
		.notice-box {
			.van-icon__image {
				width: 60px !important;
			}
		}
	}
</style>