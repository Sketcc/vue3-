<template>
	<div class="adminui-tags">
		<ul>
			<li v-for="tag in tagList" v-bind:key="tag" :class="isActive(tag) ? 'active' : '' " @contextmenu.prevent="openContextMenu($event, tag)">
				<router-link :to="tag">
				{{ tag.meta.title }}
				<i v-if="!tag.meta.affix" class="el-icon-close" @click.prevent.stop='closeSelectedTag(tag)'></i>
				</router-link>
			</li>
		</ul>
	</div>

	<transition name="el-zoom-in-top">
		<ul v-if="contextMenuVisible" :style="{left:left+'px',top:top+'px'}" class="contextmenu" id="contextmenu">
			<li @click="refreshTab()"><i class="el-icon-refresh"></i>刷新</li>
			<hr>
			<li @click="closeTabs()" :class="contextMenuItem.meta.affix?'disabled':''"><i class="el-icon-close"></i>关闭标签</li>
			<li @click="closeOtherTabs()"><i class="el-icon-folder-delete"></i>关闭其他标签</li>
			<hr>
			<li @click="screen()"><i class="el-icon-full-screen"></i>全屏当前标签</li>
			<li @click="openWindow()"><i class="el-icon-copy-document"></i>在新的窗口中打开</li>
		</ul>
	</transition>
</template>

<style>
	.contextmenu {
		position: fixed;
		margin:0;
		border-radius: 0px;
		background: #fff;
		border: 1px solid #e4e7ed;
		box-shadow: 0 2px 12px 0 rgba(0,0,0,.1);
		z-index: 3000;
		list-style-type: none;
		padding: 10px 0;
	}
	.contextmenu hr {
		margin:5px 0;
		border: none;
		height: 1px;
		font-size: 0px;
		background-color: #ebeef5;
	}
	.contextmenu li {
		margin:0;
		cursor: pointer;
		line-height: 30px;
		padding: 0 17px;
		color: #606266;
	}
	.contextmenu li i {
		font-size: 14px;
		margin-right: 10px;
	}
	.contextmenu li:hover {
		background-color: #ecf5ff;
		color: #66b1ff;
	}
	.contextmenu li.disabled {
		cursor: not-allowed;
		color: #bbb;
		background: transparent;
	}

</style>

<script>
	export default {
		name: "tags",
		data() {
			return {
				contextMenuVisible: false,
				contextMenuItem: null,
				left: 0,
				top: 0,
				tagList: this.$store.state.viewTags.viewTags
			}
		},
		props: {},
		setup() {

		},
		watch: {
			$route(e) {
				this.addViewTags(e);
			},
			contextMenuVisible(value) {
				var _this = this;
				var cm = function(e){
					let sp = document.getElementById("contextmenu");
					if(sp&&!sp.contains(e.target)){
						_this.closeMenu()
					}
				}
				if (value) {
					document.body.addEventListener('click', e=>cm(e))
				}else{
					document.body.removeEventListener('click',  e=>cm(e))
				}
			}
		},
		created() {
			this.addViewTags(this.$router.options.routes[0].children[0].children[0]);
			this.addViewTags(this.$route);
		},
		methods: {
			//增加tag
			addViewTags(route) {
				if(route.name){
					this.$store.commit("pushViewTags",route)
					this.$store.commit("pushKeepLive",route.name)
				}
			},
			//高亮tag
			isActive(route) {
				return route.path === this.$route.path
			},
			//关闭tag
			closeSelectedTag(tag) {
				this.$store.commit("removeViewTags", tag)
				this.$store.commit("removeKeepLive", tag.name)
				if (this.isActive(tag)) {
					const latestView = this.tagList.slice(-1)[0]
					if (latestView) {
						this.$router.push(latestView)
					} else {
						this.$router.push('/')
					}
				}
			},
			//tag右键
			openContextMenu(e, tag){
				this.contextMenuItem = tag;
				this.contextMenuVisible = true;
				this.left = e.clientX + 1;
				this.top = e.clientY + 1;
			},
			//关闭右键菜单
			closeMenu() {
				this.contextMenuItem = null;
				this.contextMenuVisible = false
			},
			//TAB 刷新
			refreshTab(){
				var nowTag = this.contextMenuItem;
				this.contextMenuVisible = false
				//判断是否当前路由，否的话跳转
				if(this.$route.path != nowTag.path){
					this.$router.push({
						path: nowTag.path
					})
				}
				var _this = this;
				setTimeout(function() {
					_this.$store.commit("removeKeepLive", nowTag.name)
					_this.$store.commit("setRouteShow", false)
					_this.$nextTick(() => {
						_this.$store.commit("pushKeepLive",nowTag.name)
						_this.$store.commit("setRouteShow", true)
					})
				}, 0);
			},
			//TAB 关闭
			closeTabs(){
				var nowTag = this.contextMenuItem;
				if(!nowTag.meta.affix){
					this.closeSelectedTag(nowTag)
					this.contextMenuVisible = false
				}
			},
			//TAB 关闭其他
			closeOtherTabs(){
				var nowTag = this.contextMenuItem;
				var tags = [...this.tagList];
				tags.forEach(tag => {
					if(tag.meta&&tag.meta.affix || nowTag.path==tag.path){
						return true
					}else{
						this.closeSelectedTag(tag)
					}
				})
				this.contextMenuVisible = false
			},
			//TAB 全屏标签
			screen(){
				var nowTag = this.contextMenuItem;
				this.contextMenuVisible = false
				//判断是否当前路由，否的话跳转
				if(this.$route.path != nowTag.path){
					this.$router.push({
						path: nowTag.path
					})
				}
				var element = document.getElementById('adminui-main')
				this.$TOOL.screen(element)
			},
			//新窗口打开
			openWindow(){
				var nowTag = this.contextMenuItem;
				var url = nowTag.href || '/';
				if(!nowTag.meta.affix){
					this.closeSelectedTag(nowTag)
				}
				window.open(url);
				this.contextMenuVisible = false
			}
		}
	}
</script>
