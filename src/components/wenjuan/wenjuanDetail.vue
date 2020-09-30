<template>
    <div id="wenjuan">
      <div class="head" v-if="isSubmit">
        <p class="title">{{dataInfo.title}}</p>
        <p class="sub">{{dataInfo.desc}}</p>
      </div>
      <div class="content" v-if="isSubmit">
        <div class="item" v-for="(item,index) in dataList" :key="index">
          <div>
            {{index+1}}．{{item.name}} <span v-if="item.is_required==1" style="color:red">*</span>
          </div>
          <p class="sub" v-if="item.remark">{{item.remark}}</p>
          <div v-if="item.type=='radio'"> 
            <yd-radio-group size="16" class="radio" color='#4285F4' v-model="item.value">
                <yd-radio :val="item1" v-for="(item1,index1) in item.inputList" :key="index1">{{item1}}</yd-radio>
            </yd-radio-group>
          </div>
          <div v-if="item.type=='checkbox'"> 
            <yd-checkbox-group size="16" class="checkbox" color='#4285F4' v-model="item.value">
                <yd-checkbox :val="item2" v-for="(item2,index2) in item.inputList" :key="index2">{{item2}}</yd-checkbox>
            </yd-checkbox-group>
          </div>
          <div v-if="item.type=='text'"> 
            <yd-cell-group>
              <yd-cell-item>
                <yd-input slot="right" v-model="item.value" placeholder="請輸入內容"></yd-input>
              </yd-cell-item>
            </yd-cell-group>
          </div>
          <div v-if="item.type=='textarea'"> 
            <yd-cell-group >
                <yd-cell-item>
                    <yd-textarea slot="right" v-model="item.value" placeholder="請輸入內容" ></yd-textarea>
                </yd-cell-item>
            </yd-cell-group>
          </div>
        </div>
      </div>
      <div class="content" v-if="!isSubmit">
        <p class="text_center">感謝您的參與，謝謝！</p>
      </div>
      <x-button class="Btn_Login" @click.native="submit"  text="提 交" v-if="isSubmit"></x-button>
      <x-button class="Btn_Login" @click.native="back"  text="返 回" v-if="!isSubmit"></x-button>
    </div>
</template>
<script>
  import { cookie, Loading, XButton } from 'vux'
  export default {
    components: {
      cookie,
      Loading,
      XButton
    },
    data () {
      return {
        isSubmit:true,
        dataInfo:{},
        dataList:[]
      }
    },
    computed: {
      id(){
        return this.$route.query.id
      }
    },
    created(){
      this.content_list(this.id)
    },
    mounted(){
      document.querySelector('body').setAttribute('style', 'background:#fff')
    },
    methods: {
      submit(){
        let dataList = []
        let data={}
        for(let item of this.dataList){
          if(item.is_required==1){
            if(!item.value||item.value.length==0){
              this.$dialog.toast({
                mes: '您有未填寫的題目',
                timeout: 1500,
                icon: 'error'
              })
              return false
            }else{
              dataList.push(
                {
                  id:item.id,
                  name:item.name,
                  type:item.type,
                  value:item.value,
                }
              )
            }
          }else{
            if(item.value&&item.value.length>0){
              dataList.push(
                {
                  id:item.id,
                  name:item.name,
                  type:item.type,
                  value:item.value,
                }
              )
            }
          }
        }
        data = {
          wenjuan_id:this.dataInfo.id,
          datalist:dataList
        }
        if(dataList.length==0){
          return  this.$dialog.toast({
            mes: '您填寫的問卷為空',
            timeout: 1500,
            icon: 'error'
          })
        }
        console.log('sss',data)
        this.$dialog.loading.open('正在加載中···')
        this.api.wenjuanAnswer(data).then((response)=>{
          this.$dialog.loading.close()
          if (response.data.state == 1) {
            // this.$dialog.toast({
            //   mes: '提交成功',
            //   timeout: 1500,
            //   icon: 'success'
            // })
            this.isSubmit = false
          } else {
            this.$dialog.toast({
              mes: response.data.msg,
              timeout: 1500,
              icon: 'error'
            })
          }
        })
          .catch((response)=>{
            this.$dialog.loading.close()
            this.$dialog.toast({
              mes: response.data.msg,
              timeout: 1500,
              icon: 'error'
            })
          })
      },
      back(){
        this.$router.go(-1)
      },
      content_list(id){
        this.$dialog.loading.open('正在加載中···')
        this.api.wenjuanDetail({
          id: id
        }).then((response)=>{
          this.$dialog.loading.close()
          if (response.data.state == 1) {
            let data = response.data.data
            this.dataInfo = data.data
            this.dataList = data.dataList
            .map(item=>{
              let type = ''
              if(item.type=='checkbox'){
                type=[]
              }
              if(item.type=='radio'){
                type=item.inputList[0]
              }
              return Object.assign({},item,{value:type})
            })
            console.log(this.dataList)
          } else {
            that.$dialog.toast({
              mes: response.data.msg,
              timeout: 1500,
              icon: 'error'
            })
          }
        })
          .catch((response)=>{
            that.$dialog.loading.close()
            that.$dialog.toast({
              mes: response.data.msg,
              timeout: 1500,
              icon: 'error'
            })
          })
      },
    },
    beforeCreate () {
      document.body.setAttribute('class', '')
    },
    beforeDestroy () {
      document.querySelector('body').setAttribute('style', '')
    },
  }
</script>
<style scoped lang="less" type="text/less">
    #wenjuan{
      padding: .8rem;
      font-size: .6rem;
      .head{
        text-align: center;
        .title{
          font-size: .9rem;
          padding-bottom: 10px;
        }
        .sub{
          font-size: .6rem;
        }
      }
      .content{
        .text_center{
          text-align: center;
          padding: 6rem 0;
        }
        .item{
          padding-top: 6px;
        }
        .sub{
          color:#888;
          font-size: .5rem;
          padding: 6px 0 0 0;
        }
        .radio{
          padding-top: 10px;
          padding-left: 10px;
          .yd-radio{
            display: block;
            margin-bottom: 6px;
          }
        }
        .checkbox{
          padding-top: 10px;
          padding-left: 10px;
          .yd-checkbox{
            display: block;
            margin-bottom: 6px;
          }
        }
        /deep/ .yd-cell-right{
          input[type=text]{
            font-size: 0.6rem;
            padding: 8px;
            line-height: 1.6rem;
            height: 1.6rem;
            border: 1px solid #ccc;
            margin-top: 10px;
          }
          .yd-input-clear{
            line-height: 1.6rem;
            height: 1.6rem;
          }
          .yd-input-clear:after{
            font-size: 0.8rem;
            position: relative;
            top: 6px;
          }
          .yd-input-error:after{
            font-size: 0.8rem;
            position: relative;
            top: 6px;
          }
        } 
        /deep/ .yd-textarea>textarea {
            border: 1px solid #ccc;
            padding: 8px;
            width: 100%;
            display: block;
            height: 4rem;
            font-size: .6rem;
            color: inherit;
            background-color: transparent;
        }
        /deep/ .yd-textarea-counter {
            font-size: .4rem;
            color: #b2b2b2;
            text-align: right;
            padding-top: .2rem;
        }
        /deep/ .yd-cell-box{
          margin: 0;
        }
      }
      .Btn_Login{
        margin-top: 20px;
      }
    }
</style>
