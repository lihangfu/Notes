# 基础模板

## 新建vue文件

```html
<!-- id标识vue作用的范围 -->
<div id="app">
    <!-- {{}} 插值表达式，绑定vue中的data数据 -->
    {{ message }}
</div>
<script src="vue.min.js"></script>

    // 创建一个vue对象
    new Vue({
        el: '#app',//绑定vue作用的范围
        data: {//定义页面中显示的模型数据
            message: 'Hello Vue!'
        }
    })
</script>
```

## 数据相关

1. v-bind:value或:value
    单向绑定，会自动读取data内相同变量名的值，但是自己改变时，不会影响data内的值。
2. v-model
    双向绑定，自身的值永远与data内相同变量名的值关联。
3. 取值
    {{}}两个大括号内放上data内的变量名，可以在html的text内显示。

## 事件相关

1. v-on:click或@click
    会自动执行method内的同名方法。

2. v-on:submit.prevent="onSubmit"
    阻止事件原本的默认行为，执行自定义的onSubmit方法。

## if show for
1. v-if="ok"
    会自动判断data内的ok是否为true，不为true则不执行
2. v-else
    与上一个配合使用
3. v-show="ok"
    与if功能类似，但是show会直接渲染，只是显不显示的问题，if比较懒，为真才渲染。频繁切换用show，否则用if。
4. v-for="n in 10"或v-for="(n, index) in 5"或(item, index) in userList
    根据in后面的值进行循环显示，index为关键字

# 组件模板

## 局件组键
```html
<script>
var app = new Vue({
    el: '#app',
    // 定义局部组件，这里可以定义多个局部组件
    components: {
        //组件的名字
        'Navbar': {
            //组件的内容
            template: '<ul><li>首页</li><li>学员管理</li></ul>'
        }
    }
})
</script>
```
使用组件
```html
<div id="app">
    <Navbar></Navbar>
</div>
```

## 全局组件
```html
<script>
Vue.component('Navbar', {
    template: '<ul><li>首页</li><li>学员管理</li><li>讲师管理</li></ul>'
})
</script>
```
使用组件
```html
<div id="app">
    <Navbar></Navbar>
</div>
<script src="vue.min.js"></script>
<script src="components/Navbar.js"></script>
<script>
    var app = new Vue({
        el: '#app'
    })
</script>
```

# 声明周期

**记住最常用的两个**

1. created() {}：页面渲染之前，一般用于初始化数据，加载异步数据

2. mounted() {}: 页面渲染完

```js
//===创建时的四个事件
beforeCreate() { // 第一个被执行的钩子方法：实例被创建出来之前执行
    console.log(this.message) //undefined
    this.show() //TypeError: this.show is not a function
    // beforeCreate执行时，data 和 methods 中的 数据都还没有没初始化
},
created() { // 第二个被执行的钩子方法
    console.log(this.message) //床前明月光
    this.show() //执行show方法
    // created执行时，data 和 methods 都已经被初始化好了！
    // 如果要调用 methods 中的方法，或者操作 data 中的数据，最早，只能在 created 中操作
},
beforeMount() { // 第三个被执行的钩子方法
    console.log(document.getElementById('h3').innerText) //{{ message }}
    // beforeMount执行时，模板已经在内存中编辑完成了，尚未被渲染到页面中
},
mounted() { // 第四个被执行的钩子方法
    console.log(document.getElementById('h3').innerText) //床前明月光
    // 内存中的模板已经渲染到页面，用户已经可以看见内容
},
//===运行中的两个事件
beforeUpdate() { // 数据更新的前一刻
    console.log('界面显示的内容：' + document.getElementById('h3').innerText)
    console.log('data 中的 message 数据是：' + this.message)
    // beforeUpdate执行时，内存中的数据已更新，但是页面尚未被渲染
},
updated() {
    console.log('界面显示的内容：' + document.getElementById('h3').innerText)
    console.log('data 中的 message 数据是：' + this.message)
    // updated执行时，内存中的数据已更新，并且页面已经被渲染
}
```

# 路由

## 引入路由
```html
<script src="vue.min.js"></script>
<script src="vue-router.min.js"></script>
```

## 使用路由
```html
<div id="app">
 <h1>Hello App!</h1>
    <p>
        <!-- 使用 router-link 组件来导航. -->
        <!-- 通过传入 `to` 属性指定链接. -->
        <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
        <router-link to="/">首页</router-link>
        <router-link to="/student">会员管理</router-link>
        <router-link to="/teacher">讲师管理</router-link>
    </p>
    <!-- 路由出口 -->
    <!-- 路由匹配到的组件将渲染在这里 -->
    <router-view></router-view>
</div>
```

## 监听路由刷新组件
```js
watch: {
    $route(to, from){
      this.init()
    }
  }
```


## 定义路由
```html
<script>
    // 1. 定义（路由）组件。
    // 可以从其他文件 import 进来
    const Student = { template: '<div>student list</div>' }
    const Teacher = { template: '<div>teacher list</div>' }
    // 2. 定义路由
    // 每个路由应该映射一个组件。
    const routes = [
        { path: '/', redirect: '/welcome' }, //设置默认指向的路径
        { path: '/welcome', component: Welcome },
        { path: '/student', component: Student },
        { path: '/teacher', component: Teacher }
    ]
    // 3. 创建 router 实例，然后传 `routes` 配置
    const router = new VueRouter({
        routes // （缩写）相当于 routes: routes
    })
    // 4. 创建和挂载根实例。
    // 从而让整个应用都有路由功能
    const app = new Vue({
        el: '#app',
        router
    })
    // 现在，应用已经启动了！
</script>
```

# axios

## 引入axios
```html
<script src="vue.min.js"></script>
<script src="axios.min.js"></script>
```

## 使用
```js
var app = new Vue({
    el: '#app',
    data: {
        memberList: []//数组
    },
    created() {
        this.getList()
    },
    methods: {
        getList(id) {
            //vm = this
            axios.get('http://localhost:8081/admin/ucenter/member')
            .then(response => {
                console.log(response)
                this.memberList = response.data.data.items
            })
            .catch(error => {
                console.log(error)
            })
        }
    }
})
```

