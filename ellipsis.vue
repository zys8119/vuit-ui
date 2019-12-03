<template>
	<div class="ZEllipsis">
		<div ref="A" class="ZEllipsisA">
			<slot></slot>
		</div>
		<div ref="B" class="ZEllipsisB ellipsisText" :class="{show:(show && !more)}"  :style="{'-webkit-line-clamp':row}">
			<slot></slot>
			<div class="more" v-if="show && more">···<span @click="moreClick">显示更多</span></div>
			<div class="more moreTrue" v-if="show && !more"><span @click="moreClick">收起</span></div>
		</div>
	</div>
</template>

<script>
    export default {
        name: "ellipsis",
		props:{
            row:{
                type:Number,
				default:3,
			},
		},
		data(){
            return {
                show:false,
                more:true,
                slotsText:null,
			}
		},
		mounted() {
            this.init();
        },
		methods:{
            init(){
                this.$nextTick(()=>{
                    try {
                        let h1 = this.$refs.A.clientHeight;
                        let h2 = this.$refs.B.clientHeight;
                        if(h1 > h2){
                            this.show = true;
                            return;
                        }
                        this.show = false;
                    }catch (e) {
						//err
                    }

				});
			},
            moreClick(){
                this.more = !this.more;
			}
		},
		updated() {
            this.slotsText = this.$slots.default[0].text;
        },
		watch:{
            slotsText(){
                this.init();
			}
		}
    }
</script>

<style scoped lang="less">
.ZEllipsis{
	position: relative;
	overflow: hidden;
	.ZEllipsisA{
		opacity: 0;
		position: absolute;
		left: 0;
		top: 0;
		width: 100%;
		z-index: 1;
	}
	.ZEllipsisB{
		position: relative;
		z-index: 2;
		width: 100%;
		&.ellipsisText{
			overflow: hidden;
			text-overflow: ellipsis;
			display: -webkit-box;
			word-wrap: break-word;
			word-break: break-all;
			white-space: normal !important;
			-webkit-box-orient: vertical;
			&.show{
				overflow: initial;
				text-overflow: inherit;
				display: block;
			}
		}
		.more{
			position: absolute;
			right: 0;
			bottom: 0;
			background-color: #ffffff;
			span{
				color: #33ACF2;
				cursor: pointer;
				margin-left: 5px;
			}
			&.moreTrue{
				position: relative;
				display: inline-block;
			}
		}
	}

}
</style>
