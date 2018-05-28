<template>
  <div>
    <ul class="figure-list" style="padding-top: 15px">
      <li v-for="(img, index) in imgRes" :key="index">  
        <img class="previewer-demo-img"
             :id="'image'+index"
             :src="img.src"  
             @click="clickImg(index)">
        <div :class="img.isUpload ? 'previewer-demo-bg0' : 'previewer-demo-bg1' "
             @click="clickImg(index)"
        >未上传
        </div>
      </li>
    </ul>
    <div style="padding: 10px 8px;">
      <x-button text="选择图片" @click.native="chooseImg" type="primary" :disabled="disabled"></x-button>
      <x-button text="上传图片" @click.native="uploadImage" type="warn" :disabled="disabled"></x-button>
    </div>
    <!-- 预览图片 -->
    <div v-transfer-dom>
      <previewer :list="imgRes" ref="previewer" :options="options"></previewer>
    </div>
    <!-- 图片按钮弹窗-->
    <div v-transfer-dom>
      <popup v-model="showImage" :height="imagePopupHeight" is-transparent>
        <div style="width: 100%; background-color:#fff; margin:0 auto; border-radius:5px;">
          <div style="padding:25px 15px;">
            <x-button type="primary" @click.native="showPreview()">预览</x-button>
            <x-button v-if="!disabled" type="warn" @click.native="deleteConfirm()">删除</x-button>
            <x-button @click.native="showImage = false">取消</x-button>
          </div>
        </div>
      </popup>
    </div>
    <!-- 确认框 -->
    <div v-transfer-dom>
      <confirm v-model="showConfirm" title="操作提示" @on-confirm="onConfirm" is-transparent>
        <p style="text-align:center;">确定删除图片</p>
      </confirm>
    </div>
  </div>
</template>

<script>
const IMAGE_CACHE_EXP_TIME = 4 * 86400

import Vue from 'vue'
import querystring from 'querystring'

const IMAGE_MANAGE = {
  uploadImg: '/extapiserver/eas/yhsp/image/upload',
  deleteImg: '/extapiserver/eas/yhsp/image/deleteImg',
  queryBase64Img: '/extapiserver/eas/yhsp/image/queryBase64Img',
  queryImg: '/extapiserver/eas/yhsp/image/queryImg'
}

  import {
    Previewer,
    Tab,
    TabItem,
    Flexbox,
    FlexboxItem,
    XTextarea,
    Icon,
    XImg,
    Tabbar,
    TabbarItem,
    ViewBox,
    Group,
    GroupTitle,
    Cell,
    Popup,
    XInput,
    XButton,
    Confirm,
    TransferDom
  } from 'vux'

  export default {
    directives: {
      TransferDom
    },
    components: {
      Previewer,
      Tab,
      TabItem,
      Flexbox,
      FlexboxItem,
      XTextarea,
      Icon,
      XImg,
      Confirm,
      Tabbar,
      TabbarItem,
      ViewBox,
      Group,
      GroupTitle,
      Cell,
      Popup,
      XInput,
      XButton
    },
    props: {
      // 单据的bosType
      bosType: {
        default: ''
      },
      billId: {
        default: ''
      },
      disabled: {
        default: true
      }
    },
    data () {
      return {
        imgFold: false,
        contentList: true,
        operateType: '',
        imgServeIds: [],
        imgRes: [],
        currentImgId: '',
        currentImgIndex: '',
        confirmMsg: '',
        showConfirm: false,
        itemId: '',
        remarks: '',
        isSaved: false,
        checkCertBill: '',
        itemName: '',
        itemSelect: false,
        showLeft: false,
        labelPosition: '',
        bizType: '',
        showImage: false,
        billinfo: {
          id: ''
        },
        options: {
          getThumbBoundsFn (index) {
            // find thumbnail element
            let thumbnail = document.querySelectorAll('.previewer-demo-img')[index]
            // get window scroll Y
            let pageYScroll = window.pageYOffset || document.documentElement.scrollTop
            // optionally get horizontal scroll
            // get position of element relative to viewport
            let rect = thumbnail.getBoundingClientRect()
            // w = width
            return {x: rect.left, y: rect.top + pageYScroll, w: rect.width}
            // Good guide on how to get element coordinates:
            // http://javascript.info/tutorial/coordinates
          },
          // 隐藏在Android上能显示的全屏按钮
          fullscreenEl: false
        },
        myuploadImg: this.BaseUrl + '/extapiserver/eas/yhsp/image/upload',
        mydeleteImg: this.BaseUrl + '/extapiserver/eas/yhsp/image/deleteImg',
        myqueryBase64Img: this.BaseUrl + '/extapiserver/eas/yhsp/image/queryBase64Img',
        myqueryImg: this.BaseUrl + '/extapiserver/eas/yhsp/image/queryImg'
      }
    },
    created () {
      this.wsCache.deleteAllExpires()
      // 获取图片
      this.initImg(this.billId)
    },
    computed: {
      imagePopupHeight: function () {
        // 根据是否显示删除按钮就是弹框的高度
        return this.disabled ? '140px' : '210px'
      }
    },
    methods: {
      uploadImg (bosType, imgInfo) {
        let myUploadImg = this.BaseUrl.concat(IMAGE_MANAGE.uploadImg)
        return Vue.http.post(myUploadImg,
          querystring.stringify({
            t: Vue.prototype.accessToken,
            imgInfos: JSON.stringify(imgInfo),
            bosType: bosType // 单据的BosType
          })
        )
      },
      deleteImg (billId, attachmentId) {
        let myDeleteImg = this.BaseUrl.concat(IMAGE_MANAGE.deleteImg)
        return Vue.http.get(myDeleteImg, {
          params: {
            t: Vue.prototype.accessToken || '',
            id: billId || '',
            attachmentId: attachmentId || ''
          }
        })
      },
      queryBase64Img (id) {
        let myQueryBase64Img = this.BaseUrl.concat(IMAGE_MANAGE.queryBase64Img)
        return Vue.http.get(myQueryBase64Img, {
          params: {
            t: Vue.prototype.accessToken || '',
            id: id || ''
          }
        })
      },
      queryBillImg (id) {
        let myQueryImg = this.BaseUrl.concat(IMAGE_MANAGE.queryImg)
        return Vue.http.get(myQueryImg, {
          params: {
            t: Vue.prototype.accessToken || '',
            id: id || ''
          }
        })
      },
      initImg: function (id) {
        let tempThis = this
        let successCallback = (json) => {
          let responseCode = json.data.code
          let responseMsg = json.data.msg
          let responseData = json.data.data
          if (responseCode !== 0) {
            this.showToastMsg(responseMsg)
          } else {
            if (!responseData || responseData.length === 0) {
              // this.showToastMsg('无数据')
            } else {
            }
          }
          console.log(responseData)
          for (let i in responseData) {
            // attachmentId为附件表的fid
            let easAttachmentId = responseData[i].attachmentId
            let imageid = responseData[i].imageid
            let imageObj = JSON.parse(this.wsCache.get(easAttachmentId))
            if (imageObj && imageObj.attachmentId && imageObj.src && imageObj.src.length > 0) {
              this.imgRes.push(imageObj)
            } else {
              let successCallback = (json) => {
                let responseCode = json.data.code
                let responseMsg = json.data.msg
                let responseData = json.data.data
                if (responseCode !== 0) {
                  tempThis.showToastMsg(responseMsg)
                } else {
                  if (responseData.length > 0) {

                    let imageObj = {
                      'src': 'data:image/png;base64,' + responseData,
                      'isUpload': true,
                      'imgId': imageid,
                      'attachmentId': easAttachmentId,
                      'state': 'FINISH'
                    }
                    this.wsCache.set(easAttachmentId, JSON.stringify(imageObj), {'exp': IMAGE_CACHE_EXP_TIME})
                    tempThis.imgRes.push(imageObj)
                  }
                }
              }
              let errorCallback = (json) => {
                console.log('fail')
              }
              // api.ImageManage.
              this.queryBase64Img(easAttachmentId).then(successCallback, errorCallback)
            }
          }
        }
        let errorCallback = (json) => {
          console.log('fail')
        }
        // api.ImageManage.
        this.queryBillImg(id).then(successCallback, errorCallback)
      },
      showPreview () {
        this.showImage = false
        this.$refs.previewer.show(this.currentImgIndex)
      },
      clickImg: function (index) {
        this.showImage = true
        this.currentImgIndex = index
      },
      chooseImg: function (event) {
        let wx = this.$wechat
        let imgResTemp = this.imgRes
        let tmpVue = this
        wx.chooseImage({
          count: 9, // 默认9
          sizeType: ['original', 'compressed'], // 可以指定是原图还是压缩图，默认二者都有
          sourceType: ['album', 'camera'], // 可以指定来源是相册还是相机，默认二者都有
          success: function (res) {
            if (tmpVue.isiOS) {
              // iOS需要根据localId获取图片的base64来显示
              tmpVue.getLocalImageBase64Data(res)
            } else {
              // var localIds = res.localIds // 返回选定照片的本地ID列表，localId可以作为img标签的src属性显示图片
              for (var i in res.localIds) {
                var localId = res.localIds[i]
                // Android的src保存图片localId, iOS的src保存图片base64
                imgResTemp.push({'src': localId, 'isUpload': false, 'state': 'unUpload', 'imgLocalId': localId})
              }
            }
          }
        })
      },
      // 仅iOS支持的接口！！！
      getLocalImageBase64Data: function (res) {
        let tmpImgRes = this.imgRes
        for (var i in res.localIds) {
          let localId = res.localIds[i]
          this.$wechat.getLocalImgData({
            localId: localId,
            success: function (imgData) {
              let image = imgData.localData
              image = image.replace('jpg', 'png')
              // let base64Data = 'data:image/png;base64,' + image // localData是图片的base64数据，可以用img标签显示
              // iOS的src保存图片base64,Android的src保存图片localId
              tmpImgRes.push({'src': image, 'isUpload': false, 'state': 'unUpload', 'imgLocalId': localId})
            }
          })
        }
      },
      uploadImage: function (event) {
        var thisTemp = this
        var billId = this.billId
        var bosType = this.bosType
        var imgList = this.imgServeIds
        if (typeof (billId) === 'undefined') {
          thisTemp.showToastMsg('请提交单据后再上传')
          return
        }
        if (billId === null || billId.length === 0) {
          thisTemp.showToastMsg('请提交单据后再上传')
          return
        }
        var uploadingCount = 0
        var uploadCount = 0
        for (var k in this.imgRes) {
          if (this.imgRes[k].state === 'uploading') {
            uploadingCount = uploadingCount + 1
          }
        }
        if (uploadingCount > 0) {
          thisTemp.showToastMsg('还有' + uploadingCount + '张图片在上传')
          return
        }
        for (var h in this.imgRes) {
          if (this.imgRes[h].state === 'unUpload') {
            uploadCount = uploadCount + 1
          }
        }
        if (uploadCount === 0) {
          if (this.imgRes.length === 0) {
            thisTemp.showToastMsg('请选择上传图片')
          } else {
            thisTemp.showToastMsg('图片已经全部上传成功，不用再进行上传')
          }
          return
        }
        var count = 0
        // 标记正在上传的图片
        for (var j in this.imgRes) {
          if (!this.imgRes[j].isUpload) {
            this.imgRes[j].state = 'uploading'
            count = count + 1
          }
        }
        // 循环图片数组上传图片
        var backData = []
        for (let i in this.imgRes) {
          let tempImgRes = this.imgRes[i]
          let localId = this.imgRes[i].imgLocalId
          if (this.imgRes[i].isUpload) {
            continue
          }
          this.$wechat.uploadImage({
            localId: localId, // 需要上传的图片的本地ID，由chooseImage接口获得
            isShowProgressTips: 1, // 默认为1，显示进度提示
            success: function (res) {
              imgList.push(res.serverId) // 返回图片的服务器端ID
              // var base64 = thisTemp.getImgBase64('image' + i)
              // alert('转化为base64是否出错')
              // wc.set(res.serverId, base64, {exp: 86400})
              let successCallback = (json) => {
                var responseCode = json.data.code
                var responseMsg = json.data.msg
                var responseData = json.data.data
                if (responseCode !== 0) {
                  thisTemp.showToastMsg(responseMsg)
                } else {
                  if (!responseData || responseData.length === 0) {
                    // thisTemp.showToastMsg('无数据')
                  } else {
                  }
                }
                backData.push({'src': localId, 'imgId': res.serverId, 'imgLocalId': localId})
              }
              let errorCallback = (json) => {
                tempImgRes.state = 'unUpload'
                thisTemp.showToastMsg('服务器异常')
              }
              // api.ImageManage.
              this.uploadImg(bosType, [{
                'boid': billId,
                'imageid': res.serverId
              }]).then(successCallback, errorCallback)
            },
            fail: function (res) {
              tempImgRes.state = 'unUpload'
              thisTemp.showToastMsg('图片上传失败')
            }

          })
        }
        var timesInterval = setInterval(function () {
          if (backData.length === count) {
            // for (var y in thisTemp.imgRes) {
            //   thisTemp.imgRes[y].isUpload = true
            //   thisTemp.imgRes[y].state = 'FINISH'
            // }
            for (var x in backData) {
              for (var y in thisTemp.imgRes) {
                if (backData[x].imgLocalId === thisTemp.imgRes[y].imgLocalId) {
                  thisTemp.imgRes[y].imgId = backData[x].imgId
                  thisTemp.imgRes[y].isUpload = true
                  thisTemp.imgRes[y].state = 'FINISH'
                }
              }
            }
            thisTemp.showToastMsg('图片上传成功')
            clearInterval(timesInterval)
          }
        }, 2000)
      },
      showToastMsg (msg, verticalAlign = 'middle') {
        this.$vux.toast.text(msg, verticalAlign)
      },
      getImgBase64 (imgId) {
        alert(imgId)
        var bannerImg = document.getElementById(imgId)
        bannerImg.setAttribute('crossOrigin', 'Anonymous')
        var canvas = document.createElement('canvas')
        canvas.width = bannerImg.width
        canvas.height = bannerImg.height
        var ctx = canvas.getContext('2d')
        ctx.drawImage(bannerImg, 0, 0)
        var dataURL = canvas.toDataURL('image/png')
        console.log(dataURL)
        return dataURL
      },
      onConfirm () {
        this.deleteImage(this.currentImgIndex)
      },
      deleteImage (index) {
        if (this.imgRes[index].isUpload) {
          let successCallback = (json) => {
            console.log('删除图片：', json)
            var responseCode = json.data.code
            var responseMsg = json.data.msg
            // var responseData = json.data.data
            this.editInfo = json.data.data
            this.isSaved = true
            if (responseCode !== 0) {
              this.showToastMsg(responseMsg)
            } else {
              this.showToastMsg('删除成功')
              this.imgRes.splice(index, 1)
            }
          }
          let errorCallback = (json) => {
            this.showToastMsg('图片删除失败')
          }
          // api.ImageManage.
          this.deleteImg(this.billId, this.imgRes[index].attachmentId).then(successCallback, errorCallback)
        } else {
          this.showToastMsg('删除成功')
          this.imgRes.splice(index, 1)
        }
      },
      deleteConfirm () {
        this.showImage = false
        if (!this.disabled) {
          this.showConfirm = true
        } else {
          this.showToastMsg('非保存状态下图片不可以删除')
        }
      }
    }
  }
</script>
<style scoped>/*此css只应用于本组件及其一级子组件：https://vue-loader.vuejs.org/zh/guide/scoped-css.html*/
  .previewer-demo-bg0 {
    display: none;
  }

  .previewer-demo-bg1 {
    position: relative;
    background-color: black;
    opacity: 0.6;
    z-index: 2;
    width: 100%;
    height: 50vw;
    left: 0;
    top: -50vw;
    font-size: 28px;
    line-height: 48vw;
    text-align: center;
    letter-spacing: 3px;
    color: white;
  }

  .figure-list {
    margin: 0;
    padding: 0px 5px 0px 5px;
  }

  .figure-list:after {
    content: "";
    display: block;
    clear: both;
    height: 0;
    overflow: hidden;
    visibility: hidden;
  }

  .figure-list li {
    list-style: none;
    float: left;
    overflow: hidden;
    width: 48%;
    height: 48vw;
    padding: 0 1% 1%;
  }

  .previewer-demo-img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
</style>
