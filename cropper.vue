<template>
	<div>
		<div class="cropper-content">
			<div class="cropper">
				<vueCropper
					ref="cropper"
					:img="option.img"
					:outputSize="option.size"
					:outputType="option.outputType"
					:info="true" :full="option.full"
					:canMove="option.canMove"
					:canMoveBox="option.canMoveBox"
					:original="option.original"
					:autoCrop="option.autoCrop"
					:autoCropWidth="option.autoCropWidth"
					:autoCropHeight="option.autoCropHeight"
					:fixedBox="option.fixedBox"
					@realTime="realTime"
					@imgLoad="imgLoad"
				></vueCropper>
			</div>
			<div class="show-preview" :style="{'width': previews.w + 'px', 'height': previews.h + 'px',  'overflow': 'hidden', 'margin': '5px'}" v-if="preview">
				<div :style="previews.div" class="preview">
					<img :src="previews.url" :style="previews.img">
				</div>
			</div>
		</div>

		<div class="footer-btn">
			<div class="scope-btn">
				<label class="btn" for="uploads">选择图片</label>
				<input type="file" id="uploads" style="position:absolute; clip:rect(0 0 0 0);" accept="image/png, image/jpeg, image/gif, image/jpg" @change="uploadImg($event, 1)">
				<el-button plain @click="changeScale(1)" icon="el-icon-plus"></el-button>
				<el-button plain @click="changeScale(-1)" icon="el-icon-minus"></el-button>
				<el-button plain @click="rotateLeft">
					<span style="font-weight: 600;">↺</span>
				</el-button>
				<el-button plain @click="rotateRight">
					<span style="font-weight: 600;">↻</span>
				</el-button>
			</div>
			<div class="upload-btn">
				<el-button type="primary" style="margin-left: 46px;" @click="finish('blob')">上传头像</el-button>
				<el-button type="primary" style="margin-left: 46px;" @click="down('blob')">下载头像</el-button>
			</div>
		</div>
	</div>
</template>
<script>
	import { VueCropper } from 'vue-cropper'
	// import { uploadAvatar } from '@/api/user'
	export default {
		props: {
			preview: {
				type: Boolean,
				default: true
			}
		},
		data() {
			return {
				fileName: '', // 文件名
				crap: false,
				previews: {},
				option: {
					img: 'http://img1.vued.vanthink.cn/vued751d13a9cb5376b89cb6719e86f591f3.png',//裁切图片的地址
					outputSize: 0.6,//裁剪生成图片的质量 0.1-1
					outputType: 'png',//裁剪生成图片的格式
					full: false,//是否输出原图比例的截图
					original: false,//上传图片按照原始比例渲染
					canScale: true,//图片是否允许滚轮缩放
					canMoveBox: true,//截图框能否拖动
					canMove: true,//上传图片是否可以移动
					autoCrop: true,//是否默认生成截图框
					autoCropWidth: 200,//默认生成截图框宽度
					autoCropHeight: 200,//默认生成截图框高度
					fixedBox: false,//固定截图框大小 不允许改变
					fixed: true,//开启截图框宽度和高度固定比例
					fixedNumber: [1, 1]//截图框宽高比例
				},
				downImg: '#'
			}
		},
		components: {
			VueCropper
		},
		methods: {
			changeScale(num) {
				num = num || 1;
				this.$refs.cropper.changeScale(num);
			},
			rotateLeft() {
				this.$refs.cropper.rotateLeft();
			},
			rotateRight() {
				this.$refs.cropper.rotateRight();
			},
			finish(type) {
				// 输出
				// let test = window.open('about:blank')
				// test.document.body.innerHTML = '图片生成中..'
				if (type === 'blob') {
					let formData = new FormData();
					this.$refs.cropper.getCropBlob(data => {
						let img = window.URL.createObjectURL(data);
						this.model = true;
						this.modelSrc = img;
						formData.append('file', data, this.fileName);
						console.log(formData.has('file'));
						//注意要请求的格式要是  config.headers['Content-Type'] = 'multipart/form-data'
						// uploadAvatar(formData).then(v => {
						// 	console.log('xcq', v);
						// })
					})
				} else {
					this.$refs.cropper.getCropData(data => {
						this.model = true;
						this.modelSrc = data;
					})
				}
			},
			// 实时预览函数
			realTime(data) {
				this.previews = data;
			},
			down(type) {
				// event.preventDefault()
				let aLink = document.createElement('a')
				aLink.download = 'author-img';
				// 输出
				if (type === 'blob') {
					this.$refs.cropper.getCropBlob(data => {
						console.log(data);
						this.downImg = window.URL.createObjectURL(data);
						// aLink.download = this.downImg;
						console.log(this.downImg);
					})
				} else {
					this.$refs.cropper.getCropData(data => {
						this.downImg = data;
						// aLink.href = data;
						// aLink.click();
						// 将头像图片数据发送给后台
					})
				}
			},
			uploadImg(e, num) {
				// 上传图片
				// this.option.img
				let file = e.target.files[0];
				this.fileName = file.name;
				if (!/\.(gif|jpg|jpeg|png|bmp|GIF|JPG|PNG)$/.test(e.target.value)) {
					alert('图片类型必须是.gif,jpeg,jpg,png,bmp中的一种');
					return false;
				}
				let reader = new FileReader();
				reader.onload = e => {
					let data;
					if (typeof e.target.result === 'object') {
						// 把Array Buffer转化为blob 如果是base64不需要
						data = window.URL.createObjectURL(new Blob([e.target.result]));
					} else {
						data = e.target.result;
					}
					if (num === 1) {
						this.option.img = data;
					} else if (num === 2) {
						this.example2.img = data;
					}
				}
				// 转化为base64
				reader.readAsDataURL(file);
				// 转化为blob
				// reader.readAsArrayBuffer(file)
			},
			imgLoad(msg) {
				console.log(msg);
			}
		}
	}
</script>
<style lang="scss">
	.cropper-content {
		display: flex;
		display: -webkit-flex;
		justify-content: flex-end;
		-webkit-justify-content: flex-end;

		.cropper {
			width: 350px;
			height: 300px;
		}

		.show-preview {
			flex: 1;
			-webkit-flex: 1;
			display: flex;
			display: -webkit-flex;
			justify-content: center;
			-webkit-justify-content: center;

			.preview {
				overflow: hidden;
				border-radius: 50%;
				border: 1px solid #cccccc;
				background: #cccccc;
				margin-left: 40px;
			}
		}
	}

	.footer-btn {
		margin-top: 30px;
		display: flex;
		display: -webkit-flex;
		justify-content: flex-end;
		-webkit-justify-content: flex-end;

		.scope-btn {
			width: 350px;
			display: flex;
			display: -webkit-flex;
			justify-content: space-between;
			-webkit-justify-content: space-between;
		}

		.upload-btn {
			flex: 1;
			-webkit-flex: 1;
			display: flex;
			display: -webkit-flex;
			justify-content: center;
			-webkit-justify-content: center;
		}

		.btn {
			outline: none;
			display: inline-block;
			line-height: 1;
			white-space: nowrap;
			cursor: pointer;
			-webkit-appearance: none;
			text-align: center;
			-webkit-box-sizing: border-box;
			box-sizing: border-box;
			outline: 0;
			margin: 0;
			-webkit-transition: 0.1s;
			transition: 0.1s;
			font-weight: 500;
			padding: 10px 15px;
			font-size: 12px;
			border-radius: 3px;
			color: #fff;
			background-color: #67c23a;
			border-color: #67c23a;
		}
	}
</style>
