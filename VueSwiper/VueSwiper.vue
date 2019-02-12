<!-- 
	time:2018.10.10
	name:Vue轮播横竖向插件
	description:通过配置一些参数完成基本的轮播效果<vueswiper :config="argments"></vueswiper>
	arguments:{
		type:'row',//row代表横向，col代表竖向，默认是row
		imgDatas:[
			'img/swiper1.jpg',
			'img/swiper2.jpg',
			'img/swiper3.jpg',
			'img/swiper4.jpg'
		],//图片的路径，以数组的形式存在，默认是空数组[],必须存在
		width:"100%",//整体的宽度，可以写百分比可以写px，默认是500px宽度
		height:'500px',//整体的高度，可以写百分比可以写px，默认是300px高度
		auto:true,//是否自动播放，false表示不播放，true表示播放，默认为true
		btnLocation:'inside',//按钮是在内部还是外部，inside表示内部显示，outside表示外部显示，默认是内部
		timeInterval:2000,//轮播的时间间隔，单位毫秒，默认是2000ms
		timeAnimation:1000,//切换图片的动画时间，单位毫秒，默认是1000ms
	}
 -->
<template>
	<div class="vueswiper" :style="{width:config.width,height:config.height}">
		<div v-if="config.type=='row'" class="vueswiperArrow vueswiperArrowlt" @click="lt">&lt;</div>
		<div v-if="config.type=='row'" class="vueswiperArrow vueswiperArrowgt" @click="rt">&gt;</div>
		<div v-if="config.type=='col'" class="vueswiperArrowCol vueswiperArrowColTop" @click="lt"></div>
		<div v-if="config.type=='col'" class="vueswiperArrowCol vueswiperArrowColBottom" @click="rt"></div>
		<div class="vueswiperImgCon">
			<div v-if="config.type=='row'" class="vueswiperCon" :style="{width:config.imgDatas.length*100+'%',left:moveLeft+'%'}">
				<img v-for="(item,index) in config.imgDatas"
					:src="item"
					:alt="item"
					:style="{width:100/config.imgDatas.length+'%',height:'100%'}"
					/>
			</div>
			<div v-if="config.type=='col'" class="vueswiperConCol" :style="{height:config.imgDatas.length*100+'%',top:moveLeft+'%'}">
				<img  v-for="(item,index) in config.imgDatas"
					:src="item"
					:alt="item"
					:style="{height:100/config.imgDatas.length+'%',width:'100%'}"
				/>
			</div>
		</div>
		<div :class="{vueswiperBtn:config.btnLocation=='inside'&&config.type=='row',
					vueswiperBtn2:config.btnLocation=='outside'&&config.type=='row',
					vueswiperBtnCol:config.btnLocation=='inside'&&config.type=='col',
					vueswiperBtnCol2:config.btnLocation=='outside'&&config.type=='col'}">
			<div :class="{vueswiperBtnCon:config.type=='row',vueswiperBtnConCol:config.type=='col'}">
				<i v-for="(i,index) in config.imgDatas.length"
					:class="{vueswiperIndex:config.type=='row',vueswiperIndexCol:config.type=='col',vueswiperIndexChoose:index==btnIndex}"
					v-show="index>0&&index<config.imgDatas.length-1"
					@click="chooseImg(index)"
				></i>
			</div>
		</div>
	</div>
</template>
<script type="text/javascript">
	export default {
		props:{
			config:{
				type:Object,
				required:true
			}
		},
		data(){
			return {
				moveLeft:-100,//向左移动的距离
				chooseIndex:1,//选中的序号
				btnIndex:1,//按钮的序号
				orientation:'',//轮播的方向
				carouselInterval:null,//轮播计时器
				autoInterval:null,//自动播放
			}
		},
		methods:{
			lt(){
				this.stop();
				clearInterval(this.carouselInterval);
				this.chooseIndex<=0?this.chooseIndex=0:this.chooseIndex--;
				this.chooseIndex==0?this.btnIndex=this.config.imgDatas.length-2:this.btnIndex=this.chooseIndex;
				this.move();
			},
			rt(){
				this.stop();
				clearInterval(this.carouselInterval);
				this.chooseIndex>=this.config.imgDatas.length-1?this.chooseIndex=this.config.imgDatas.length-1:this.chooseIndex++;
				this.chooseIndex==this.config.imgDatas.length-1?this.btnIndex=1:this.btnIndex=this.chooseIndex;
				// 移动到了最后一个按钮
				this.move();
			},
			chooseImg(index){
				if(index!=this.chooseIndex){
					this.stop();
					clearInterval(this.carouselInterval);
					this.chooseIndex=index;
					this.btnIndex=index;
					this.move();
				}
			},
			move(){
				var terminus = 0-this.chooseIndex*100;//最终图片移动到的位置
				var distanceInterval = (terminus-this.moveLeft)/60
				var animationInterval=this.config.timeAnimation/60;//60帧是人眼顺畅的感受
				this.carouselInterval=setInterval(()=>{
					//向右移动
					if(distanceInterval<0){
						//超出终点了
						if(this.moveLeft+distanceInterval<terminus){
							this.moveLeft=terminus;
							clearInterval(this.carouselInterval);
							this.start();
							//是不是最后一张
							if(this.chooseIndex==this.config.imgDatas.length-1){
								//最后一张图,移动完成后要处理图片的位置
								this.moveLeft=-100;
								this.chooseIndex=1;
							}
						}else{
							this.moveLeft+=distanceInterval;
						}
					}
					//向左移动
					else{
						//超出终点了
						if(this.moveLeft+distanceInterval>terminus){
							this.moveLeft=terminus;
							clearInterval(this.carouselInterval);
							this.start();
							//是不是第一张
							if(this.chooseIndex==0){
								this.moveLeft=0-(this.config.imgDatas.length-2)*100;
								this.chooseIndex=this.config.imgDatas.length-2;
							}
						}else{
							this.moveLeft+=distanceInterval;
						}
					}
				},animationInterval)
			},
			start(){
				if(this.config.auto){
					this.autoInterval=setInterval(()=>{
						this.rt();
					},this.config.timeInterval);
				}
			},
			stop(){
				if(this.config.auto){
					clearInterval(this.autoInterval)
				}
			}
		},
		mounted(){
			// 参数的配置
				// 轮播类型
				if(this.config.type && typeof this.config.type=='string'){

				}else{
					this.config.type='row';
				}
				// 图片资源
				if(this.config.imgDatas && typeof this.config.imgDatas=='object' && this.config.imgDatas.length>0){
						//原来的图片上加第一张图为初始图片的最后一张，原来的图片上加最后一张图为初始图片的第一张，用于轮播处理
						var fristImg=this.config.imgDatas[0],
							lastImg=this.config.imgDatas[this.config.imgDatas.length-1];
						this.config.imgDatas.push(fristImg);
						this.config.imgDatas.unshift(lastImg);
				}else{
					this.config.imgDatas=[];
				}
				// 框的宽
				if(this.config.width && typeof this.config.width=='string'){

				}else{
					this.config.width='500px';
				}
				// 框的高
				if(this.config.height && typeof this.config.height=='string'){

				}else{
					this.config.height='300px';
				}
				// 轮播的间隔时间
				if(this.config.timeInterval && typeof this.config.timeInterval=='number'){

				}else{
					this.config.timeInterval=2000;
				}
				// 轮播的动画时间
				if(this.config.timeAnimation && typeof this.config.timeAnimation=='number'){

				}else{
					this.config.timeAnimation=1000;
				}
				//是否自动播放
				if(typeof this.config.auto=='boolean'){

				}else{
					this.config.auto=true;
				}
				//按钮的位置
				if(this.config.btnLocation && typeof this.config.btnLocation=='string'){

				}else{
					this.config.btnLocation='inside';
				}
			//自动播放
			this.start();
		}
	}
</script>
<style type="text/css">
	ul,ol{list-style: none;}
	.vueswiper{
		position: relative;
		user-select: none;
		-webkit-user-select:none;
		-o-user-select:none;
		-moz-user-select:none;
	}
	.vueswiperArrow{
		position: absolute;
		display: flex;
		justify-content: center;
		align-items: center;
		height: 100%;
		width: 10%;
		max-width: 80px;
		z-index: 500;
		color: #fff;
		font-size: 20px;
		cursor: pointer;
		opacity: 0;
		transition: all 0.3s;
		}.vueswiperArrowlt{
			left:0px;
			top: 0px;
			background: -webkit-linear-gradient(0deg,rgba(0,0,0,0.5), rgba(0,0,0,0)); /* Safari 5.1 - 6.0 */
			background: -o-linear-gradient(0deg,rgba(0,0,0,0.5), rgba(0,0,0,0)); /* Opera 11.1 - 12.0 */
			background: -moz-linear-gradient(0deg,rgba(0,0,0,0.5), rgba(0,0,0,0)); /* Firefox 3.6 - 15 */
			background: linear-gradient(90deg,rgba(0,0,0,0.5), rgba(0,0,0,0)); /* 标准的语法 */
		}.vueswiperArrowgt{
			top: 0px;
			right:0px;
			background: -webkit-linear-gradient(0deg,rgba(0,0,0,0), rgba(0,0,0,0.5)); /* Safari 5.1 - 6.0 */
			background: -o-linear-gradient(0deg,rgba(0,0,0,0), rgba(0,0,0,0.5)); /* Opera 11.1 - 12.0 */
			background: -moz-linear-gradient(0deg,rgba(0,0,0,0), rgba(0,0,0,0.5)); /* Firefox 3.6 - 15 */
			background: linear-gradient(90deg,rgba(0,0,0,0), rgba(0,0,0,0.5)); /* 标准的语法 */
		}.vueswiperArrow:hover{
			opacity: 1;
		}
		/*竖向箭头*/
		.vueswiperArrowCol{
			position: absolute;
			width:100%;
			height:20%;
			max-height:50px;
			cursor: pointer;
			z-index: 500;
			opacity: 0;
			transition: all 0.3s;
			}.vueswiperArrowColTop{
				top: 0;
				left: 0;
				background: -webkit-linear-gradient(90deg,rgba(0,0,0,0.5), rgba(0,0,0,0)); /* Safari 5.1 - 6.0 */
				background: -o-linear-gradient(90deg,rgba(0,0,0,0.5), rgba(0,0,0,0)); /* Opera 11.1 - 12.0 */
				background: -moz-linear-gradient(90deg,rgba(0,0,0,0.5), rgba(0,0,0,0)); /* Firefox 3.6 - 15 */
				background: linear-gradient(180deg,rgba(0,0,0,0.5), rgba(0,0,0,0)); /* 标准的语法 */
			}.vueswiperArrowColBottom{
				bottom: 0;
				left: 0;
				background: -webkit-linear-gradient(-90deg,rgba(0,0,0,0.5), rgba(0,0,0,0)); /* Safari 5.1 - 6.0 */
				background: -o-linear-gradient(-90deg,rgba(0,0,0,0.5), rgba(0,0,0,0)); /* Opera 11.1 - 12.0 */
				background: -moz-linear-gradient(-90deg,rgba(0,0,0,0.5), rgba(0,0,0,0)); /* Firefox 3.6 - 15 */
				background: linear-gradient(0deg,rgba(0,0,0,0.5), rgba(0,0,0,0)); /* 标准的语法 */
			}.vueswiperArrowCol:hover{
				opacity: 1;
			}
	/*图片内容*/
	.vueswiperImgCon{
		position: relative;
		width: 100%;
		height:100%;
		overflow: hidden;
		}.vueswiperCon{
			position:absolute;
			top: 0;
			height: 100%;
		}.vueswiperConCol{
			position: absolute;
			left: 0;
			width:100%;
		}.vueswiperConCol>img{
			display: block;
		}
	/*按钮区域*/
	.vueswiperBtn{
		position: absolute;
		left: 0px;
		bottom: 20px;
		width: 100%;
		display: flex;
		justify-content:center;
		}.vueswiperBtn2{
			position: absolute;
			left: 0px;
			bottom: -20px;
			width: 100%;
			display: flex;
			justify-content:center;
		}
		.vueswiperBtnCon{
			display: flex;
			align-items: center;
			}.vueswiperIndex{
				border-radius: 50%;
				border:1px solid #000;
				width: 10px;
				height: 10px;
				background-color: transparent;
				margin: 0 10px;
				transition: all 0.3s;
				cursor: pointer;
			}.vueswiperIndex:hover{
				background-color: rgba(0,0,0,0.5);
			}.vueswiperIndexChoose{
				background-color: #000 !important;
			}
		/*竖向按钮*/
		.vueswiperBtnCol{
			position: absolute;
			height:100%;
			right: 20px;
			top: 0;
			display: flex;
			align-items: center;
		}.vueswiperBtnCol2{
			position: absolute;
			height:100%;
			right: -20px;
			top: 0;
			display: flex;
			align-items: center;
			}.vueswiperBtnConCol{
			display: flex;
			flex-direction: column;
			}.vueswiperIndexCol{
				border-radius: 50%;
				border:1px solid #000;
				width: 10px;
				height: 10px;
				background-color: transparent;
				margin: 5px 0;
				transition: all 0.3s;
				cursor: pointer;
			}.vueswiperIndexCol:hover{
				background-color: rgba(0,0,0,0.5);
			}
</style>