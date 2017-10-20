<template>
  <div id="app">
    <view-box>
        <x-header
                slot="header"
        :left-options="{showBack:false}"
        >
            <div slot="left">直播</div>
            <div>网易</div>
            <div slot="right">搜索</div>
        </x-header>
            <sc :lock-y="true">
                <div class="tab">
                    <tab>
                        <tab-item selected>推荐</tab-item>
                        <tab-item>要闻</tab-item>
                        <tab-item>游戏</tab-item>
                        <tab-item>科技</tab-item>
                        <tab-item>娱乐</tab-item>
                        <tab-item>体育</tab-item>

                    </tab>
                </div>
            </sc>
        <scroller
                class="my-scroller"
                :on-refresh="refresh"
                :onInfinite="infinite"
                ref="myRef"
        >
            <Swiper
                    :list="SwiperList"
                    v-model="swiperIndex"
                    :loop="true"

            >

            </Swiper>

            <marquee>
                <marquee-item v-for="i in marqueelist">
                    {{i.title}}
                </marquee-item>
            </marquee>

            <panel
                    :list="datalist">
            </panel>
            <panel
                    :list="moreDataList">
            </panel>

        </scroller>
        <tabbar slot="bottom">
            <tabbar-item>
                <img src="./assets/ico1.png" slot="icon">
                <span slot="label">首页</span>
            </tabbar-item>
            <tabbar-item>
                <img src="./assets/ico3.png" slot="icon">
                <span slot="label">推荐</span>
            </tabbar-item>
            <tabbar-item>
                <img src="./assets/ico4.png" slot="icon">
                <span slot="label">我的</span>
            </tabbar-item>
        </tabbar>
    </view-box>
    <router-view></router-view>
  </div>
</template>

<script>
    import {ViewBox,XHeader,Tabbar, TabbarItem,Tab, TabItem,Scroller as sc,Swiper,Marquee, MarqueeItem,Panel } from 'vux';
    var refreshkey=['A','B01','B02','B03','B04','B05','B06','B07','B08','B09','B010'];
    var key=0;
    var start=0;
    var end=start+9;
    var keyValue ='A';
    var initloaded=false;
    function getRefreshKey() {

        key++
        if(key===refreshkey.length){
            key=0;
        }
        keyValue=refreshkey[key]

    }
export default {
  name: 'app',
    data(){
      return{
          SwiperList:[],
          swiperIndex:0,
          marqueelist:[],
          datalist:[],
          moreDataList:[]
      }
    },
    components:{
        ViewBox,
        XHeader,
        Tabbar, TabbarItem,
        Tab, TabItem,
        sc,
        Swiper,
        Marquee, MarqueeItem,
        Panel
    },
    created(){
        this.$jsonp('http://3g.163.com/touch/jsonp/sy/recommend/0-9.html').then(data=>{
            console.log(data)
            this.SwiperList=data.focus.filter(item=>{
                return item.addata===null
            }).map(item=>{
                return {
                    url:item.link,
                    img:item.picInfo[0].url,
                    title:item.title
                }
            });
            this.marqueelist=data.live.filter(item=>{
                return item.addata===null
            }).map(item=>{
                return {

                    title:item.title
                }
            });
            this.datalist=data.list.filter(item=>{
                return item.addata===null
            }).map(item=>{
                return {
                    src:item.picInfo[0].url,
                    title:item.title,
                    desc:item.digest,
                    url:item.link

                }
            })
            initloaded=true;
        })
    },
    methods:{
        refresh(){
            getRefreshKey();
            this.$jsonp('http://3g.163.com/touch/jsonp/sy/recommend/0-9.html',{
                miss:'00',
                refresh:keyValue

            }).then(data=>{
                this.datalist=data.list.filter(item=>{
                    return item.addata===null
                }).map(item=>{

                    return {
                        src:item.picInfo[0].url,
                        title:item.title,
                        desc:item.digest,
                        url:item.link

                    }
                });
                this.$refs.myRef.finishPullToRefresh();
                this.$vux.toast.show('刷新完成')
            })
        },
        infinite(){
            if (!initloaded){
                this.$refs.myRef.finishInfinite();
                return
            }
            console.log(2)
            this.$jsonp('http://3g.163.com/touch/jsonp/sy/recommend/${start}-${end}.html',{
                miss:'00',
                refresh:keyValue

            }).then(data=>{
                this.moreDataList=data.list.filter(item=>{
                    return item.addata===null
                }).map(item=>{

                    return {
                        src:item.picInfo[0].url,
                        title:item.title,
                        desc:item.digest,
                        url:item.link

                    }
                });
                start +=10;
                end=start + 9;
                this.$refs.myRef.finishInfinite();
            })
        }
    }
}
</script>

<style lang="less">
  @import "~vux/src/styles/reset.less";
html,body{
    margin:0;
    height: 100%;
    width:100%;
    overflow: hidden;
}
    #app{
        height:100%;
        .vux-header{
            position: relative;
            top:0;
            left:0;
            z-index: 1;
        }
        .vux-tab{


        }
        .tab{
            width:600px;
        }
        .weui-media-box__hd, .weui-media-box__hd img{
            width:102px;
            height: 78px;
        }
        .weui-media-box__bd{
            height: 78px;
        }
        .my-scroller{
            top:90px;
        }
        .loading-layer{
            padding-bottom: 90px;
        }
    }

</style>
