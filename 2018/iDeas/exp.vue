<!--
作者：chenguanchao
时间：2018年6月22日
内容：模版页
-->
<template>
  <div class="pagebox">
    <div class="pagewin">

      <!-- log按钮，用于打印data或做一些你想做的事 -->
      <div class="testConsole" @click="testConsole">log</div>

      <!-- header部分 -->
      <div class="headerbox" v-if="show.header">
        <div class="headerwin">
          <!-- <div class="headersearch">
            <input type="" name="">
          </div> -->
          <div class="headerbar">
            <!-- <div class="headerbutton">button</div> -->
          </div>
        </div>
      </div>

      <!-- list部分 -->
      <div class="listbox" v-if="show.List">
        <div class="listwin">
          <div class="listitem" v-for="">
            
          </div>
        </div>
      </div>

      <!-- edit部分 -->
      <div class="editbox" v-if="show.Edit">
        <div class="editwin">
          <group>
            <datetime title="计划日期" :readonly="saveEdit || eliminateEditing" clear-text="今天" @on-clear="setToday" v-model="plandate" text-align="right" placeholder="请选择" ></datetime>
            <group style="padding-bottom:10px">
                <x-textarea placeholder="请输入备注" :max="80" :readonly="saveEdit" v-model="billinfo.remark" :show-clear="false"></x-textarea>
            </group>
            <image-component ref="imgcomp" :billId="billId" :bosType="'1DCD11FC'" :uploadImageTips="'请保存单据后再上传图片'" :disabled="saveEdit" ></image-component>
          </group>
          
        </div>
      </div>

      <!-- searchF7部分 -->
      <div class="searchbox" v-if="show.search">
        <div class="searchwin">
          <div class="searchheader"></div>
          <div class="searchlist">
            <div class="searchlistitem" v-for=""></div>
          </div>
        </div>
      </div>

      <!-- alert部分 -->
      <div class="alertbox" v-if="show.alert">
        <div class="alertwin">
          <p></p>
          <button></button>
        </div>
      </div>

      <!-- popup部分 -->
      <div class="popupbox" v-if="show.popup">
        <div class="popupwin">
          <popup height="100%" :show-mask="false" position="top">
            <search
            :results="search.result"
            :placeholder="search.placeholder"
            v-model="search.keyword"
            position="absolute"
            auto-scroll-to-top
            @on-change="search_getResult"
            @result-click="search_resultClick"
            @on-submit="search_onSubmit"
            @on-cancel="search_onCancel"
            @on-focus="search_onFocus"
            ref="search"
            ></search>
          </popup>  
        </div>
      </div>

      <!-- 空数据 -->
      <div class="emptybox" v-if="show.Empty">
        <div class="emptywin">
          <img src="../assets/img/empty.svg"/>
          <p>抱歉暂无数据</p>
        </div>
      </div>

      <!-- 蒙版 -->
      <div class="maskbox" v-if="show.Mask">
        <div class="maskwin">
          <p></p>
        </div>
      </div>

      <!-- toolbar部分 -->
      <div class="toolbarbox" v-if="show.toolbar">
        <div class="toolbarwin">
          <div class="headerbutton">button</div>
        </div>
      </div>
    
    </div>
  </div>
</template>

<script>
  import {
    // Group,
    // Alert,
    // Cell,
    // Search,
    // debounce,
    // XInput,
    // Tabbar,
    // Popup,
    // TabbarItem,
    // Tab,
    // ViewBox,
    // Swiper,
    // SwiperItem,
    // XDialog,
    // XHeader,
    // dateFormat,
    // Datetime,
    // XTable,
    // Checklist,
    // Confirm,
    // TabItem
  } from 'vux'
  import imageCompoment from '../components/ImageCompoment.vue'
  // import CacheKeys from '../assets/js/cache-key.js'
  // import { ICONBACK, ICONSAVE, ICONSUBMIT, ICONDELETE } from '../assets/js/constant.js'
  import api from '../api'
  export default {
    components: {
      // Group,
      // Alert,
      // Cell,
      // Search,
      // debounce,
      // XInput,
      // Tabbar,
      // Popup,
      // TabbarItem,
      // Datetime,
      // Tab,
      // ViewBox,
      // Swiper,
      // SwiperItem,
      // XDialog,
      // XHeader,
      // dateFormat,
      // XTable,
      // Checklist,
      // Confirm,
      // TabItem,
      'image-component': imageCompoment
    },
    data () {
      return {
        self: { // 只限本页面使用的变量，比较随意
          name: 'exp'
        },
        state: { // 页面状态罗列，通过watch此部分来切换页面状态
          node: 'audit', // 列出所有备选状态，如(new/save/audit/finish)
          inputs: { // 各种非空、非法判断在此登记
            name: false,
            email: false
          }
        },
        transplate: {  // 之后可能需移植/被移植的功能数据
          module1: {} // 建议长命名，并在其数据、方法依赖上注释移植来源，便于后续项目移植开发
                      // module1 @exp.vue 2018-06-21
        },
        show: { // 各部分显示与否，一般通过changesShow方法登记并操作
          page: 0,
          List: true,
          Edit: true,
          Empty: false,
          popup: false,
          alert: false,
          Search: false,
          Mask: false,
          header: true,
          toolbar: true
        },
        perform: { // 当前展示的数据
          title: 'title',
          // list: [listItem1],
          note: '1',
          item1: '',
          item2: ''
        },
        save: { // 当前本地储存数据，可以创建子项作为提交目录，也可以用于类似添加分录、草稿、购物车功能
          // list: [listItem1, listItem2],
          note: '12'
        },
        edit: { // 当前本地编辑中数据，编辑页面最常用的部分
          // listItem: {listItem3},
          input1: '123',
          input2: {
            id: '',
            name: '',
            result: ''
          }
        },
        search: {
          placeholder: '',
          keyword: '',
          result: []
        },
        jsons: { // 返回的需要复用的json数据统一储存
          json1: {
            key1: '', // 开发阶段如有必要就注释一份json各key值的列表，直观
            key2: ''
          },
          json2: {}
        }
        // sessionStorage: {}, // 先在data声明再storage储存
        // localStorage: {}
      }
    },
    // watch: {
    //   state: function (a) {
    //     console.log('state' , this.state, a)
    //     this.state.page = 1
    //   },
    //   edit: function (b) {
    //     console.log('edit' , this.edit, b)
    //   }
    // },
    created: {},
    methods: {
      /* show部分 */
      show_change(scene) { // 页面的变化入口 统一在此处声明各场景名字来控制
        if (scene === 'empty') {
          this.show.empty = true
          this.show.list = false
          this.show.edit = false
        } else if (scene === 'search') {
          this.show.search = !this.show.search
        } else if (scene === 'popup') {
          this.show.popup = !this.show.popup
          this.show.mask = this.show.popup || this.show.alert
        } else if (scene === 'alert') {
          this.show.alert = !this.show.alert
          this.show.mask = this.show.popup || this.show.alert
        } else {
          console.log('Fail at show_change, scene undefined')
        }
      },
      /* show部分 end */

      /* edit部分 */
      /* edit部分 end */


      json_getJson1 () { // 一般ajax函数
        let successCallback = (json) => {
          this.jsons.json1 = json.data.data
          let responseData = this.jsons.json1
          console.log(responseData)
        }
        let errorCallback = (json) => {
        }
        api.Base.getDeliverEntryF7('123').then(successCallback, errorCallback)
      },

      /* search部分 */
      search_open (val) {
        if (val === 'STORAGE') {
          this.$refs.storage.blur()
          this.search.placeholder = '请输人库存组织编码或名称'
        } else if (val === 'ELIMINATE') {
          this.$refs.eliminate.blur()
          this.searchF7.searchPlaceholder = '请输入淘汰申请编码或名称'
        }
        this.isOpenF7 = true
        window.scrollTo(0, 0)
        this.searchF7.searchType = val
        this.$refs.search.onFocus()
      },
      search_getResult: debounce(
        function (val) {
          let successCallback = (json) => {
            if (json.data.code === 0) {
              let requestData = json.data.data
              let rs = []
              if (this.searchF7.searchType === 'STORAGE' || this.searchF7.searchType === 'TRANTYPE') {
                for (let i = 0; i < requestData.length; i++) {
                  rs.push({
                    id: requestData[i].id,
                    title: requestData[i].name
                  })
                }
              } else if (this.searchF7.searchType === 'BREEDINGBATCH') {
                for (let i = 0; i < requestData.length; i++) {
                  rs.push({
                    id: requestData[i].id,
                    title: requestData[i].name,
                    name: requestData[i].simpleName
                  })
                }
              } else if (this.searchF7.searchType === 'PIGGRADE') {
                for (let i = 0; i < requestData.length; i++) {
                  rs.push({
                    id: requestData[i].id,
                    title: requestData[i].name,
                    displayname: requestData[i].displayname
                  })
                }
              } else if (this.searchF7.searchType === 'ELIMINATE') {
                for (let i = 0; i < requestData.length; i++) {
                  let dateString = this.yhsp.getLocalTime(requestData[i].bizdate)
                  console.log(this.searchF7.searchValue)
                  rs.push({
                    id: requestData[i].id,
                    title: requestData[i].breedingbatch_name,
                    date: dateString,
                    number: requestData[i].number
                  })
                  // console.log(dateString)
                }
              }
              this.searchF7.searchResults = rs.sort(function (a, b) { return a.number > b.number })
            }
          }
          let errorCallback = (json) => {
            this.$vux.toast.text('请求失败')
          }
          if (this.searchF7.searchType === 'STORAGE') {
            api.Base.getStorageOrgUnit(this.searchF7.searchValue).then(successCallback, errorCallback)
          } else if (this.searchF7.searchType === 'BREEDINGBATCH') {
            api.Base.getPigBatchById(this.billinfo.storageorgunitid, this.searchF7.searchValue).then(successCallback, errorCallback)
          } else if (this.searchF7.searchType === 'TRANTYPE') {
            api.Base.getTranType('SYF004', '', this.searchF7.searchValue).then(successCallback, errorCallback)
          } else if (this.searchF7.searchType === 'PIGGRADE') {
            api.Base.getPigGrade().then(successCallback, errorCallback)
          } else if (this.searchF7.searchType === 'ELIMINATE') {
            api.InvSaleStorage.eliminateApply(this.timeStamp - 1728000000, this.timeStamp, this.billinfo.breedingbatchid, 40, this.searchF7.searchValue).then(successCallback, errorCallback)
          }
        }
      ),
      search_onCancel () {
        this.isOpenF7 = false
        this.searchF7.searchPlaceholder = ''
        this.searchF7.searchResults = []
        if (this.searchF7.searchType === 'PIGGRADE') {
          this.showEntry = true
        }
      },
      search_onFocus () {
        this.getResult('')
      },
      search_onSubmit () {
        this.$refs.search.setBlur()
      },
      search_resultClick (item) {
        if (this.searchF7.searchType === 'STORAGE') {
          this.billinfo.storageorgunitid = item.id
          this.billinfo.storageorgunit_name = item.title
          this.billinfo.breedingbatchid = ''
          this.billinfo.breedingbatch_name = ''
          this.setCache()
        } else if (this.searchF7.searchType === 'BREEDINGBATCH') {
          this.billinfo.breedingbatchid = item.id
          this.billinfo.breedingbatch_name = item.name
        } else if (this.searchF7.searchType === 'TRANTYPE') {
          this.billinfo.trantypeid = item.id
          this.billinfo.trantype_name = item.title
        } else if (this.searchF7.searchType === 'PIGGRADE') {
          this.entryItem.piggradeid = item.id
          this.entryItem.piggrade_name = item.title
          this.entryItem.piggrade_displayname = item.displayname
          this.showEntry = true
        } else if (this.searchF7.searchType === 'ELIMINATE') {
          this.billinfo.eliminateId = item.id
          this.billinfo.eliminateName = item.title
          this.billinfo.eliminateDate = item.date
          this.billinfo.eliminateNumber = item.number
          this.eliminateEdit()
        }
        this.searchF7.searchValue = ''
        this.searchF7.searchResults = []
        this.isOpenF7 = false
      }
      /* search部分 end */
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

    .testConsole{
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

    .headerbox{
      .headerwin{
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 50px;
        z-index: 2;
        background-color: #ff8848;
      }
    }

    .toolbarbox{
      .toolbarwin{
        position: fixed;
        bottom: 0;
        left: 0;
        width: 100%;
        height: 50px;
        z-index: 2;
        background-color: red;
      }
    }

    .listbox{
      .listwin{
        position: relative;
        width: 100%;
        height: 100%;
        padding-top: 50px;
        padding-bottom: 50px;
      }
    }

    .editbox{
      editwin{
        position: relative;
        width: 100%;
        height: 100%;
        padding-top: 50px;
        padding-bottom: 50px;
      }
    }

    .popupbox{
      .popupwin{
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 50px;
        z-index: 2;
        background-color: red;
      }
    }

  }
}
</style>
