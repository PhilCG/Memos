<!--
作者：chenguanchao
时间：2018年7月17日
内容：经营变更模版页
-->
<template>
  <div class="pagebox">
    <div class="pagewin">
      
      <!-- log按钮，用于打印data或做一些你想做的事 -->
      <div class="testlog" @click="testLog">log</div>

      <group>
        <x-input placeholder="请选择" text-align="right" :show-clear="false" readonly @click.native="search_open('SALEATTRIBUTE')">
          <span slot="label">申请部门<font  color="red"> *</font></span>
        </x-input>
      </group>
      <div>
        <hr>
        <p>{{perform.page.name}}!</p>
        <p>{{perform.page.id}}?</p>
        <hr>
      </div>

      <!-- search! -->
      <div height="100%" v-if="show.search">
        <search
        ref="search"
        v-model="search.keyword"
        position="absolute"
        auto-scroll-to-top
        :placeholder="search.now.placeholder"
        :results="search.list"
        @on-change="search_getList"
        @result-click="search_resultClick"
        @on-submit="search_submit"
        @on-cancel="search_cancel"
        @on-focus="search_focus"
        ></search>
      </div>
      <!-- search! end -->
    </div>
  </div>
</template>

<script>
  import {
    Group,
    Search,
    debounce,
    // Popup
    XInput
  } from 'vux'
  // import imageCompoment from '../components/ImageCompoment.vue'
  // import CacheKeys from '../assets/js/cache-key.js'
  // import { ICONBACK, ICONSAVE, ICONSUBMIT, ICONDELETE } from '../assets/js/constant.js'
  import api from '../api'
  export default {
    components: {
      Group,
      // // Alert,
      // // Cell,
      Search,
      debounce,
      XInput
      // Tabbar,
      // Popup,
      // TabbarItem,
      // Datetime,
      // Tab,
      // // ViewBox,
      // Swiper,
      // SwiperItem,
      // Checklist,
      // GroupTitle,
      // // // XDialog,
      // // // XHeader,
      // dateFormat,
      // PopupRadio,
      // XTextarea,
      // // XTable,
      // // // Checklist,
      // // // Confirm,
      // TabItem
      // 'image-component': imageCompoment
    },
    data () {
      return {
        show: {
          search: false
        },
        perform: { // 当前展示的数据，一般页面内容直接放在这里
          page: {
            name: '2',
            id: '1'
          }
        },
        search: {
          keyword: '',
          list: [], // 候选列表展示名称
          // json: {
          //   data: {
          //     data: {}
          //   }
          // }, // 候选json数据暂存
          now: { // 这里存放从map拿到的搜索词对应的字段目录
            type: '',
            placeholder: '',
            keyNumber: 0,
            ajax: function () {},
            passResult: function () {}
            // jsonIndex: function () {},
            // jsonKey: function () {},
            // localKey: function () {}
          },
          map: { // 维护这个数据来完成各种搜索框的增查改删
            SALEATTRIBUTE: { // 搜索词
              type: 'SALEATTRIBUTE',
              placeholder: '请选择销售类型',
              ajax: (keyword, successCallback, errorCallback) => {
                let success = (json) => {
                  let ajaxList = []
                  let resData = json.data.data
                  for (let i = 0; i < resData.length; i++) {
                    ajaxList.push({
                      // 首项默认title，此为搜索列表显示名称，其它左边命名key值时要与保存赋值位置的key名对应
                      title: resData[i].name,
                      name: resData[i].name,
                      id: resData[i].id,
                      number: resData[i].number
                    })
                  }
                  successCallback(ajaxList, json)
                }
                let error = (json) => { errorCallback(json) }
                api.Base.getSaleAttributeInfo('', keyword).then(success, error)
              },
              passResult: (ajaxListItem, condition) => {
                if (condition) {
                  this.perform.page.name = ajaxListItem.name
                  this.perform.page.id = ajaxListItem.id
                  this.perform.page.number = ajaxListItem.number
                }
              }
              // 默认下列json的key值直接从json.data.data目录取到，否则在此处声明
              // jsonIndex: function () {
              //   return window.eval('json.data.data')
              // },
              // 拿到json后 将按这里key值顺序取值，第一项默认设置为显示名称
              // jsonKey: function (json, i) {
              //   let Json = json || {}
              //   console.log(Json)
              //   const JsonKey = [
              //     '.name',
              //     '.id'
              //   ]
              //   return window.eval('Json' + JsonKey[i])
              // },
              // // 然后点选列表项会按这里顺序一一填值
              // localKey: function (local, i) { // this.perform.page.saleAttribute
              //   let Local = local || {}
              //   console.log(Local)
              //   const LocalKey = [
              //     '.name',
              //     '.id'
              //   ]
              //   return window.eval('Local' + LocalKey[i])
              // }
            }
          }
        }
      }
    },
    created: function () {
      // this.setHeight()
    },
    methods: {
      // myEval (str) {
      //   let Fn = Function // 一个变量指向Function，防止有些前端编译工具报错
      //   return new Fn('return ' + str)()
      // },
      testLog () {
        this.search_resultClick()
        console.log(this.search)
        console.log(this.perform)
      },
      show_change (scene, state) { // 页面的变化入口 统一在此处的scene声明各场景，state来控制开关
        if (scene === 'empty') {
          this.show.empty = true
          this.show.list = false
          this.show.edit = false
        } else if (scene === 'search') {
          this.show.search = state
        } else {
          console.log('Fail at show_change, scene undefined')
        }
      },
      /* search! */
      // api.Base.listSaleType().then(successCallBack, errorCallBack)
      search_open (searchType) {
        let thisMap = this.search.map[searchType]
        console.log('thisMap', thisMap)
        if (thisMap !== undefined) {
          this.search.now = thisMap
          console.log('this.search.now', this.search.now)
          this.show_change('search', true)
          this.search_getList()
          // this.$refs.search.focus()
          setTimeout(() => {
            this.$refs.search.onFocus()
          }, 1)
          // this.$refs.search.search_focus()
          // this.search_focus()
          window.scrollTo(0, 0)
        }
      },
      search_getList: debounce(
        function () {
          let successCallback = (ajaxList, json) => {
            // console.log('searchArray', searchArray)
            this.search.list = ajaxList
            /* jshint ignore:start */
            // this.search.list = eval('return ' + this.search.now.jsonIndex)
            /* jshint ignore:end */
            // this.search.json = json
            // // let aa = window.eval(this.arr)
            // let aa = window.eval(this.search.now.jsonIndex)
            // this.search.list = aa
            // this.search.json = json.data.data
            // let a = this.search.json
            // for (let i = 0; i < this.search.now.keyNumber; i++) {
            //   this.search.list.push(this.search.now.jsonKey(a[i], 0))
            // }
            // console.log(a)
            // this.search.list = this.search.now.jsonIndex
            // this.search.list = this.myEval('return ' + this.search.now.jsonIndex)
            // this.search.list = eval('json.data.data' + '.' + this.search.now.jsonIndex)
            // if (json.data.code === 0) {
          }
          let errorCallback = (json) => {
            this.$vux.toast.text('请求失败')
          }
          // let searchAjax = this.search.now.ajax()
          // searchAjax(this.search.keyword, successCallback, errorCallback)
          this.search.now.ajax(this.search.keyword, successCallback, errorCallback)
        }
      ),
      // func_eval (a, b) {
      //   return window.eval('(' + a + '=' + b + ')')
      // },
      search_cancel () {
        this.search_reset()
        this.show_change('search', false)
      },
      search_focus () {
        this.search_getList('')
      },
      search_submit () {
        this.$refs.search.setBlur()
      },
      search_resultClick (item) {
        // for (let i = 0; i < this.search.keyNumber; i++) {
        //   let thisJsonKey = this.search.now.jsonKey(this.search.json, i)
        //   let thisLocalKey = this.search.now.localKey(this.perform.page.saleAttribute, i)
        //   // let thisLocalKey = this.myEval(this.search.now.localKey[i])
        //   console.log('thisLocalKey', thisLocalKey + ':' + 'thisJsonKey', thisJsonKey)
        //   thisLocalKey = thisJsonKey
        // }
        console.log('item', item + 'this.search.type', this.search.type)

        // 没有特殊运行要求的直接传值就好，有特殊要求的就做一下判断
        if (this.search.now.type === 'SALEATTRIBUTE') {
          this.search.now.passResult(item, true)
          // this.perform.page.name = item.name
          // this.perform.page.id = item.id
          // this.perform.page.number = item.number
          // console.log('result', this.perform.page)
        // } else if () {
        } else {
          this.search.now.passResult(item, true)
        }
        this.search_reset()
        this.show_change('search', false)
      },
      search_reset () {
        this.search.keyword = ''
        this.search.list = []
        this.search.now = {
          type: '',
          placeholder: '',
          ajax: function () {},
          passResult: function () {}
          // jsonIndex: function () {},
          // jsonKey: function () {},
          // localKey: function () {}
        }
      }
      /* search! end */
    }
  }
</script>
<style lang="less" scoped>
.pagebox{
  width: 100%;
  height: 100%;

  .pagewin{
    width: 100%;
    height: 100%;

    .testlog{
      position: absolute;
      top: 25px;
      left: 25px;
      min-width: 25px;
      min-height: 25px;
      background-color: red;
      z-index: 10000;
      text-align: center;
      line-height: 25px;
      color: white;
    }
  }
}
</style>
