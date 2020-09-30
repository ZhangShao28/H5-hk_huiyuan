<template>
  <div id="news">
    <div class="news_content">
      <yd-infinitescroll :callback="loadList" ref="infinitescrollDemo">
        <yd-list theme="1" slot="list">
          <div
            v-for="(item, index) in wenjuan_list"
            :key="index"
            class="wenjuan_list"
            @click="go(item)"
          >
            <div class="box">
              <div class="wenjuan_title">
                <div class="title">
                  <div class="tit">{{ item.title }}</div>
                  <div class="">{{ item.end_time }}</div>
                </div>
              </div>
              <div class="ic_more" v-if="item.type == 1">
                <img src="/static/img/ic_more.png" alt="" />
              </div>
            </div>
          </div>
        </yd-list>
        <span slot="loadingTip">正在加載···</span>
        <span slot="doneTip" v-show="show_more">沒有更多數據了~~</span>
      </yd-infinitescroll>
      <div v-show="show_nodata" class="no_data">
        <img src="/static/img/ic_apply_invalid.png" alt="" />
        <div class="title">暫無數據</div>
      </div>
    </div>
  </div>
</template>
<script>
import {
  Tab,
  TabItem,
  Checklist,
  Sticky,
  cookie,
  Loading,
  LoadMore,
  Panel
} from "vux";
export default {
  components: {
    Tab,
    TabItem,
    Checklist,
    Sticky,
    Panel,
    cookie,
    Loading,
    LoadMore
  },
  data() {
    return {
      //        aurl:'',
      scrollTop: "",
      page: 1,
      pageSize: 10,
      wenjuan_list: [],
      show_nodata: false,
      show_more: true,
      loading_show: false //按钮禁用
    };
  },
  mounted() {
    document.title = "征集";
    console.log("title", document.title);
    // document.title=cookie.get('xh_name')
    //      document.addEventListener('scroll',this.handelscroll)
    //      let  htp = location.href.split('//')[0], host = window.location.host
    //      this.aurl = htp + '//' + host + '/NewsDetail?id='
    document
      .getElementsByTagName("body")[0]
      .setAttribute("style", "background:#f5f5f5");
    var that = this;
    that.$refs.infinitescrollDemo.$emit("ydui.infinitescroll.reInit");
    that.show_nodata = false;
    that.show_more = false;
    that.wenjuan_list = [];
    that.page = 1;
    this.loadList();
  },
  methods: {
    go(item) {
      if (item.type != 0) {
        this.$router.push({
          path: "/wenjuanDetail",
          query: {
            id: item.id
          }
        });
      }
    },
    onImgError(item, $event) {
      console.log(item, $event);
    },
    loadList() {
      var that = this;
      that.$dialog.loading.open("正在加載中···");
      that.api
        .wenjuanIndex({
          page: this.page,
          cid: cookie.get("cid")
        })
        .then(
          response => {
            console.log(response);
            if (response.data.state == 1) {
              // sucess callback
              var data = response.data.data.data_list;
              that.$dialog.loading.close();
              for (var i = 0; i < data.length; i++) {
                that.wenjuan_list.push(data[i]);
              }
              if (response.data.data.pages <= 2 && data.length == 0) {
                that.show_nodata = true;
              }
              if (response.data.data.pages <= 2 && data.length <= 10) {
                that.show_more = false;
              }
              if (
                data.length < that.pageSize ||
                that.page == response.data.data.pages
              ) {
                /* 所有数据加载完毕 */
                that.$refs.infinitescrollDemo.$emit(
                  "ydui.infinitescroll.loadedDone"
                );
                if (response.data.data.pages >= 3) {
                  that.show_more = true;
                } else {
                  that.show_more = false;
                }
                return;
              }

              /* 单次请求数据完毕 */
              this.$refs.infinitescrollDemo.$emit(
                "ydui.infinitescroll.finishLoad"
              );
              that.page++;
            } else {
              that.$dialog.loading.close();
              that.$dialog.toast({
                mes: response.data.msg,
                timeout: 1500,
                callback: () => {}
              });
            }
          },
          response => {
            // error callback
            that.$dialog.loading.close();
            console.log(response);
          }
        );
    },
    handler(el) {
      var that = this;
      that.$refs.infinitescrollDemo.$emit("ydui.infinitescroll.reInit");
      that.show_nodata = false;
      that.show_more = false;
      that.wenjuan_list = [];
      that.page = 1;
      that.loadList();
    }
  },
  beforeCreate() {
    document.body.setAttribute("class", "");
  },
  beforeDestroy() {
    document.getElementsByTagName("body")[0].setAttribute("style", "");
  }
};
</script>
<style lang="less" type="text/less">
#news {
  .vux-tab-wrap {
    box-shadow: 0px 1px 3px rgba(0, 0, 0, 0.1);
    .vux-tab .vux-tab-item.vux-tab-selected {
      color: #3d82d9;
      border-bottom: 3px solid #3d82d9;
    }
    .vux-tab-ink-bar {
      background-color: #3d82d9;
      width: 70px;
      margin: 0 auto;
    }
    .vux-tab .vux-tab-item {
      font-size: 16px;
    }
  }
  .yd-list {
    border-radius: 8px;
  }
  .yd-list-donetip {
    padding: 0 0 0.4rem 0;
    font-size: 0.6rem;
    color: #999;
  }

  .yd-list-loading {
    padding: 0 0 0.5rem 0;
    font-size: 0.6rem;
  }
}
#news .news_content {
  .no_data {
    text-align: center;
    padding-top: 1.5rem;
    color: #999;
    width: 100%;
    font-size: 0.65rem;
    .title {
      position: relative;
      top: -40px;
    }

    img {
      width: 100%;
    }
  }
}
</style>
<style scoped lang="less" type="text/less">
#news {
  .news_content {
    /*padding: 55px 15px 70px 15px;*/
    padding: 10px 10px 70px 10px;

    .wenjuan_list {
      border-radius: 8px;
      box-shadow: 0px -1px 10px rgba(0, 0, 0, 0.1);
      background: #fff;
      margin: 0 auto 10px auto;
      padding: 10px;
      color: #333;
      .box {
        display: flex;
        justify-content: space-between;
        align-items: center;
        .ic_more {
          img {
            width: 20px;
            height: 20px;
          }
        }
      }
      .wenjuan_title {
        .logo {
          float: left;
          width: 40px;
          height: 40px;
          background: #ccc;
          border-radius: 25px;
          margin-right: 10px;

          img {
            width: 40px;
            height: 40px;
            border-radius: 25px;
          }
        }
        .title {
          font-size: 0.5rem;
          color: #999;

          .tit {
            font-size: 0.65rem;
            color: #666;
            margin-bottom: 8px;
          }
        }
      }
      .news_con {
        width: 13.2266rem;
        margin: 0 auto;

        .title {
          /*font-weight: bold;*/
          font-size: 0.72rem;
          line-height: 1.2rem;
          color: #333;
          margin: 8px 0;
          text-align: justify;
          width: 100%;
        }

        /*.title i{*/
        /*display:inline-block;*/
        /*width:100%;*/
        /*}*/
        .news_img {
          /*width: 13.2266rem;*/
          width: 100%;
          height: 100%;
          /*height: 155px;*/
          overflow: hidden;

          img {
            width: 100%;
            border-radius: 4px;
            /*height: 6.6133rem;*/
          }
        }
      }
    }
  }
}
</style>
