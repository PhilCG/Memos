本vue模版默认使用方法：在目标页面添加整合以下代码，按需修改  
1.将api中的BaseUrl挂载到vue(例如在main.js)
```
Vue.prototype.BaseUrl = BaseUrl
```
2.父页面引入模版
```
<image-component style="padding-bottom:40px"
				ref="imgcomp" :billId="billId"
				:bosType="''"
				:disabled="editState || !this.hasOpPermission">
</image-component>
```
3.父页面js添加
```
<script>
	import ImageCompoment from 'vue-wx-image'
	export default {
	    components: {
	      'image-component': ImageCompoment
	    }
	}
</script>
```