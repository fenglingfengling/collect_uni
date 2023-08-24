<script>
	export default {
		onLaunch: function() {
			/**
			 * 初始化微信云
			 * 	env 环境id,如 kkb-kn8rp  cloud://cloud-k6rjw.6b6b-cloud-k6rjw-1302447185/mdata/img/5afe7c2a4d14a.png 
			 */
			wx.cloud.init({
				env:this.globalData.envId//'cloud-k6rjw'
			});
		},
		onShow: function() {
			
		},
		onHide: function() {
			
		},
		/**
		 * 全局变量
		 * 	userInfo 用户信息
		 */
		globalData:{
			userInfo:null,
			envId:"cloud-k6rjw",
			myPath:"cloud://cloud-k6rjw.636c-cloud-k6rjw-1302747416/"
		}
	}
</script>

<style>
	/*每个页面公共css */
	
	/* 灰色分割线 */
	.line_gray {
		width: 750upx;
		height: 15upx;
		background: #f6f6f6;
		border-top: 1px solid #ddd;
	}
</style>
