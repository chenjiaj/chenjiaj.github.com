
---
title: Vue-给对象新增属性（使用Vue.$set()）
date: 2018.04.24 11:27
categories: 'vue'
tags: vue
---

在开发过程中，我们时常会遇到这样一种情况：当vue的data里边声明或者已经赋值过的对象或者数组（数组里边的值是对象）时，向对象中添加新的属性，如果更新此属性的值，是不会更新视图的。

根据官方文档定义：**如果在实例创建之后添加新的属性到实例上，它不会触发视图更新**。

当你把一个普通的 JavaScript 对象传入 Vue 实例作为 `data` 选项，Vue 将遍历此对象所有的属性，并使用 [`Object.defineProperty`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 把这些属性全部转为 [getter/setter](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Working_with_Objects#%E5%AE%9A%E4%B9%89_getters_%E4%B8%8E_setters)。

受现代 JavaScript 的限制 (以及废弃 Object.observe)，Vue 不能检测到对象属性的添加或删除。由于 Vue 会在初始化实例时对属性执行 getter/setter 转化过程，所以属性必须在 data 对象上存在才能让 Vue 转换它，这样才能让它是响应的。

看以下实例：

    <template>
         <div>
		<p @click="addd(obj)">{{obj.d}}</p>
		<p @click="adde(obj)"> {{obj.e}}</p>
	</div>
    </template>

     <script>
      export default {
          data(){
                return {
                    obj:{}
                }
          },
          mounted() {
			this.obj = {d: 0};
			this.obj.e = 0;
			console.log('after--', this.obj);
	      },
         methods: {
			addd(item) {
				item.d = item.d + 1;
				console.log('item--',item);
			},
			adde(item) {
				item.e = item.e + 1;
				console.log('item--',item);
			}
           }
      }
     </scirpt>

![image.png](/images/Vue-给对象新增属性（使用Vue-$set）-img/1.png)

可以看出d属性是有get 和 set方法的，而新增的e属性是没有的。

点击触发3次addd，点击触发3次adde,页面效果及控制台信息如下

![image.png](/images/Vue-给对象新增属性（使用Vue-$set）-img/2.png)



![image.png](/images/Vue-给对象新增属性（使用Vue-$set）-img/3.png)


此时触发1次addd,页面效果如下：

![image.png](/images/Vue-给对象新增属性（使用Vue-$set）-img/4.png)

![image.png](/images/Vue-给对象新增属性（使用Vue-$set）-img/5.png)


由此可以看出，更新新增属性e，是不会更新视图，但是会改变其值，当更新原有属性d时会更新视图，同时将新增的属性e的值也更新到视图里边


### 解决方案

官方定义：

Vue 不允许在已经创建的实例上动态添加新的根级响应式属性 (root-level reactive property)。然而它可以使用 Vue.set(object, key, value) 方法将响应属性添加到嵌套的对象上：

Vue.set(vm.obj, 'e', 0)
您还可以使用 vm.$set 实例方法，这也是全局 Vue.set 方法的别名：

this.$set(this.obj,'e',02)

有时你想向已有对象上添加一些属性，例如使用 Object.assign() 或 _.extend() 方法来添加属性。但是，添加到对象上的新属性不会触发更新。在这种情况下可以创建一个新的对象，让它包含原对象的属性和新的属性：

// 代替 `Object.assign(this.obj, { a: 1, e: 2 })`
this.obj= Object.assign({}, this.obj, { a: 1, e: 2 })

上述实例解决如下：

![image.png](/images/Vue-给对象新增属性（使用Vue-$set）-img/6.png)


点击触发3次addd，点击触发3次adde,页面效果及控制台信息如下：

![image.png](/images/Vue-给对象新增属性（使用Vue-$set）-img/7.png)

![image.png](/images/Vue-给对象新增属性（使用Vue-$set）-img/8.png)







