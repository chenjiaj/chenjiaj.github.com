
---
title: Vue组件-confirm
date: 2018.03.29 11:42
categories: 'vue'
tags: vue
---

对于confirm组件，我总共试过以下三种方式去实现，最后还是选择了用直接通过this调用promise方法的形式。

### 一、使用的地方引用，传入属性和方法

在需要的地方引入组件，然后传入对应的属性和方法

组件

    <template>
	<div :class="{'pop-up':true,'show':isShow}">
		<div class="popup-mask" v-if="hasMark"></div>
		<transition name="bottom">
			<div class="popup-note bottom">
				<div class="pop-content">
					<div class="pop-tit">
						{{title}}
					</div>
					<p class="pop-note hasTitle">
						<span class="msg" v-html="msg"></span>
					</p>
					<div class="btn-wrapper" v-if="type == 'alert'" @click="alertClick">
						<span class="btn btn-block yes-btn">{{alertBtnText}}</span>
					</div>
					<div class="btn-wrapper" v-if="type == 'confirm'">
						<span @touchstart="noClick" class="btn">{{noBtnText}}</span>
						<span @touchstart="yesClick" class="btn yes-btn">{{yesBtnText}}</span>
					</div>
				</div>
			</div>
		</transition>
	</div>
    </template>
    <script>
	export default {
		props: {
			isShow: {
				type: Boolean,
				default: false
			},
			title: {
				type: String,
				default: '提示'
			},
			msg: {
				type: String,
				default: ''
			},
			type: {
				type: String,
				default: 'alert'
			},
			alertBtnText: {
				type: String,
				default: '我知道了'
			},
			yesBtnText: {
				type: String,
				default: '确定'
			},
			noBtnText: {
				type: String,
				default: '取消'
			},
			hasMark: {
				type: Boolean,
				default: true
			}
		},
		methods: {
			noClick() {
				this.$emit('noClick');
			},
			yesClick() {
				this.$emit('yesClick');
			},
			alertClick() {
				this.$emit('alertClick');
			}
		}
	}
    </script>
    <style lang='less'>
	@import "../../../static/less/components/tip/index.less";
    </style>

使用

    <template>
	<div>
		<message
			type="confirm"
			:title="confirmTitle"
			:msg="confrimMsg"
			:isShow="isConfirmShow"
			yesBtnText="覆盖"
			@noClick="cancelDel"
			@yesClick="doDel">
		</message>
	</div>
    </template>

    <script>
	import Message from '../components/message/message.vue'

	export default {
		components: {
			Message
		},
		data() {
			return {
				isConfirmShow: false,
				confirmTitle: '计划冲突',
				confrimMsg: ''
			}
		},
        methods:{
                  //取消
			cancelDel() {
				this.isConfirmShow = false;
			},
			//确定
			doDel() {
				this.isConfirmShow = false;
			}
          }
	}
    </script>


这种方式的问题在于在每个使用的地方都需要引用,如果组件使用频率比较多就不适合这样写，因为项目一般是会使用懒加载的，如果使用过多可以写到公共文件里边，不需要每次使用到的地方都去加载一次。

另外 如果在遇到以下情景，使用起来也会特别的不方便。在离开路由前需要弹出一个确定框，点击确定执行next(),点击取消执行next(false)，如果按照这种方式的话需要提前定义好cancelDel和doDel方法，这个时候不好把next传递给这两个方法，只能通过一个变量或熟悉把next存起来，这样使用起来也不方便，并且阅读起来也会比较困难。

     beforeRouteLeave(to, from, next) {
	    this.$confirm({
					title: '',
					msg: '模式未保存，确定离开？',
					yesBtnText: "离开"
				}).then(() => {
					next();
				}).catch(() => {
					next(false);
				});
	},

###  二、全局引入组件，使用vuex控制组件的显示

组件代码

    <template>
	<div :class="{'pop-up':true,'show':isShow}">
		<div class="popup-mask" v-if="hasMark"></div>
		<transition name="bottom">
			<div class="popup-note bottom">
				<div class="pop-content">
					<div class="pop-tit">
						{{title}}
					</div>
					<p class="pop-note hasTitle">
						<span class="msg" v-html="msg"></span>
					</p>
					<div class="btn-wrapper" v-if="type == 'alert'" @click.stop="alertClick">
						<span class="btn btn-block yes-btn">{{alertBtnText}}</span>
					</div>
					<div class="btn-wrapper" v-if="type == 'confirm'">
						<span @click.prevent="noClick" class="btn">{{noBtnText}}</span>
						<span @click.prevent="yesClick" class="btn yes-btn">{{yesBtnText}}</span>
					</div>
				</div>
			</div>
		</transition>
	</div>
    </template>
    <script>
	import {mapState} from 'vuex'

	export default {
		computed: mapState({
			title: state => state.confirm.title,
			isShow: state => state.confirm.isShow,
			msg: state => state.confirm.msg,
			type: state => state.confirm.type,
			hasMark: state => state.confirm.hasMark,
			alertBtnText: state => state.confirm.alertBtnText,
			noBtnText: state => state.confirm.noBtnText,
			yesBtnText: state => state.confirm.yesBtnText,
		}),
		methods: {
			alertClick() {
				this.$store.commit('alertClick')
			},
			noClick() {
				this.$store.commit('noClick')
			},
			yesClick() {
				this.$store.commit('yesClick')
			}
		}
	}
    </script>

    <style lang='less' type="text/less" scoped>
	@import "../../../static/less/components/tip/index.less";
    </style>

vuex部分代码


    let yesCallBack = () => {};
    let noCallBack = () => {};
    let alertCallBack = () => {};

    export default {
	state: {
		title: '',
		isShow: false,
		msg: '',
		type: 'confirm',
		hasMark: true,
		alertBtnText: '',
		noBtnText: '取消',
		yesBtnText: '确定',
	},
	mutations: {
		confirm(state, data) {
			Object.assign(state, {
				title: data.title,
				isShow: true,
				msg: data.msg,
				type: 'confirm',
				hasMark: data.hasMark === 'undefined' ? true : data.hasMark,
				alertBtnText: data.alertBtnText,
				noBtnText: data.noBtnText || '取消',
				yesBtnText: data.yesBtnText || '确定',
			});

			let yesCb = data.yesClick;
			let noCb = data.noClick;
			if (yesCb) {
				yesCallBack = yesCb;
			} else {
				yesCallBack = () => {
				};
			}

			if (noCb) {
				noCallBack = noCb;
			} else {
				noCallBack = () => {
				};
			}
		},
		alert(state, data) {
			Object.assign(state, {
				title: data.title,
				isShow: true,
				msg: data.msg,
				type: 'alert',
				hasMark: data.hasMark === 'undefined' ? true : data.hasMark,
				alertBtnText: data.alertBtnText,
				noBtnText: data.noBtnText || '取消',
				yesBtnText: data.yesBtnText || '确定',
			});

			let alertCb = data.alertClick;
			if (alertCb) {
				alertCallBack = alertCb;
			} else {
				alertCallBack = () => {
				};
			}
		},
		noClick(state) {
			noCallBack();
			state.isShow = false;
		},
		yesClick(state) {
			yesCallBack();
			state.isShow = false;
		},
		alertClick(state) {
			alertCallBack();
			state.isShow = false;
		}
	}
    }

使用

  全局主文件写入confirm

    <template>
	<div>
		<router-view></router-view>
		<confirm></confirm>
	</div>
    </template>

    <script>
	import confirm from './components/confirm/index.vue'
	export default {
		components: {
			confirm
		}
	}
    </script>
    <style lang="less">
	@import "../static/less/base.less";
    </style>

在使用的地方写入这样调用即可

    this.$store.commit('confirm', {
				title: 'title',
				msg: '是否确认删除',
				yesClick: () => {
					console.log('yes');
				},
				noClick: () => {
					console.log('no');
				}
			});

这种方式的好处是不需要每个使用的地方都引入组件，只需要在主文件引入即可。但是这个方法写的时候需要注意，在每次调用赋值的时候需要将以前所有值清空，避免属性受上次调用影响。

这种方式相对来说比较麻烦，除了维护组件以外，还要维护vuex的状态，还需要提前把模板写入页面，相对来说比较麻烦。

### 三、直接通过this调用promise方法的形式

这种方式实用简单，阅读起来也符合语义。
并且在未调用之前

vue文件

      <template>
	<div :class="{'pop-up':true,'show':show}">
		<div class="popup-mask" v-if="hasMark"></div>
		<transition name="bottom">
			<div class="popup-note bottom">
				<div class="pop-content">
					<div class="pop-tit">
						{{title}}
					</div>
					<p class="pop-note hasTitle">
						<span class="msg" v-html="msg"></span>
					</p>
					<div class="btn-wrapper" v-if="type == 'alert'" @click.stop="alertClick">
						<span class="btn btn-block yes-btn">{{alertBtnText}}</span>
					</div>
					<div class="btn-wrapper" v-if="type == 'confirm'">
						<span @touchstart.prevent="noClick" class="btn">{{noBtnText}}</span>
						<span @touchstart.prevent="yesClick" class="btn yes-btn">{{yesBtnText}}        </span>
					</div>
				</div>
			</div>
		</transition>
	</div>
    </template>
    <script>
	export default {
		props: {
			title: {
				type: String,
				default: '提示'
			},
			msg: {
				type: String,
				default: ''
			},
			type: {
				type: String,
				default: 'alert'
			},
			alertBtnText: {
				type: String,
				default: '我知道了'
			},
			yesBtnText: {
				type: String,
				default: '确定'
			},
			noBtnText: {
				type: String,
				default: '取消'
			},
			hasMark: {
				type: Boolean,
				default: true
			}
		},
		data() {
			return {
				promiseStatus: null,
				show: false
			}
		},
		methods: {
			confirm() {
				let _this = this;
				this.show = true;
				return new Promise(function (resolve, reject) {
					_this.promiseStatus = {resolve, reject};
				});
			},
			noClick() {
				this.show = false;
				this.promiseStatus && this.promiseStatus.reject();

			},
			yesClick() {
				this.show = false;
				this.promiseStatus && this.promiseStatus.resolve();

			},
			alertClick() {
				this.show = false;
				this.promiseStatus && this.promiseStatus.resolve();
			}
		}
	}
    </script>


    <style lang='less'>
	@import "../../../static/less/components/tip/index.less";
    </style>

confirm.js

    import Vue from 'vue'
    import message from './message.vue'
    const VueComponent = Vue.extend(message);
    const vm = new VueComponent().$mount();
    let init = false;
    let defaultOptions = {
	yesBtnText: '确定',
	noBtnText: '取消'
    };

    const confirm = function (options) {
	Object.assign(vm,defaultOptions , options,{
		type:'confirm'
	});

	if (!init) {
		document.body.appendChild(vm.$el);
		init = true;
	}

	return vm.confirm();
    };

    export default confirm;

全局注册

    import confirm from './views/components/message/confirm'
    Vue.prototype.$confirm = confirm;


使用

    this.$confirm({
		title: '',
		msg: '模式未保存，确定离开？',
		yesBtnText: "离开"
	}).then(() => {
		console.log('yes')
		})
       .catch(() => {
		console.log('cancel')
	});

这种方式和第二中方式写的时候都要注意，每次调用有些值需要重置，因为都是全局的，共用同一个组件，避免不同地方调用相互影响。

第三种方式涉及的知识点

##### 1.Vue.extend()
使用基础 Vue 构造器，创建一个“子类”。参数是一个包含组件选项的对象
.vue单文件经过webpack打包之后是一个组件示例对象，因此可以传到Vue.extend中生成一个包含此组件的类

##### 2.new VueComponent().$mount()

new VueComponent() 创建实例，调用$mount()可以手动编译

如果.$mount('#app')有参数，表示手动编译并且挂载到该元素上。

##### 3.$el属性 类型：string | HTMLElement

手动编译后的示例对象中存在一个$el对象（dom元素），可以作为模板被插入到页面中。

##### 4.Vue.prototype 添加 Vue 实例方法
