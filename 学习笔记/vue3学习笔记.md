#  Vue3.x学习笔记

### Vue生命周期函数

> #### beforeCreate()
>
> **beforeCreate()**钩子是在调用 **app.mount() 创建 Vue 实例之后， createApp() 中的配置项生效之前**调用。

> ####  created()
>
> **created()** 钩子是在 **beforeCreated() 之后，createApp() 中的配置项生效之后调用**，这个时候计算属性、方法、watcher监听器已经配置完成。

> #### beforeMount()
>
> 在 created 生命周期执行完成以后，我们的**应用**还没有挂载到 app.mount() 指定的 HTML  元素上，在**挂载之前会调用 beforeMount() 生命周期钩子**，这个时候应用还没有在页面上渲染出来。

> #### mounted()
>
> mounted() **会在应用挂载到 app.mount() 指定的 HTML 元素上之后执行**，这个时候应用已经在页面上渲染出来了。

> #### beforeUpdate()
>
>  当应用中的 **HTML 模板需要重新渲染时**，例如 data 中的属性发生了变化，那么在刷新前会执行 beforeUpdate() 生命周期钩子。

> #### updated()
>
> updated() 是在**数据更新之后、HTML 重新渲染完成之后调用。**

> #### beforeUnmount()
>
> beforeUnmount() 会在**应用卸载前执行**，调用 app.unmount()会卸载组件，那么在正式卸载前，先执 行beforeUnmount() 生命周期钩子，这个时候 Vue 应用还是正常的，可以在里边做一些清理收尾操作。

> #### unmounted()
>
> unmounted() 生命周期会在**应用正式卸载之后调用**，这个时候跟应用有关的事件监听，或者指令绑定都已经卸载了。

### Vue 组件数据流向设计

> Props Down，Event Up（属性向下，事件向上）

### Mixins 问题

- 命名易冲突
- 各配置项之间的功能逻辑不能复用

**解决方案：Vue3组合式API**

### 组件错误处理：全局错误处理

   在 main.js 中配置错误函数

```js
import { createApp } from "vue";
import App from "./App.vue";

const app = createApp(App);

app.config.errorHandler = (err, vm, info) => {
  console.log(err);
  console.log(vm);
  console.log(info);
};

app.mount("#app");

```

### Composition API 优势

- **组件逻辑的复用**
- **逐步应用**
- **与 Options API 共存**
- **将配置式的代码改成了函数式的**

####  Vue 2 Options API

使用 Options API 时，属于同一功能的逻辑代码会分散在 data、methods中，查看代码时需要在不同的配置项中反复跳跃。

#### Vue 3 Composition API

- 使用 Composition API 时，组件的所有代码都会写在 setup() 的函数中，**使得功能逻辑的抽离更为方便**，弥补了使用 Mixins 复用逻辑的缺陷，但是逻辑也显得没有那
- Composition API 中**没有 beforeCreate 和 created 这两个生命周期函数**，因为 **setup 函数本来就是在这两个生命周期期间执行的**。因此这两个生命周期之间的业务逻辑可以直接写在 setup 函数中
- 可以将 setup 中的代码全部抽离到 composables  中编写，composables 逻辑越独立越好，可以减少组件文件的代码，增强复用。



#### 使用 ref() 函数替代 data() 函数

`reactive()` 会直接返回响应式的数据，只支持对象和数组类型的数据，给 `ref` 传递对象类型的数据时，在底层也会调用 `reactive` 函数，`ref` 定义的响应式数据会储存在返回值的 `value`属性中。



#### watch 和 watchEffect 函数的区别

二者都是用来监听响应式数据的，`wacth` **需要指定要监听的属性名**， 而 `watchEffect` 不需要指定监听的属性，**内部会自定监听使用到的数据**。

  

#### 使用 toRef 将 porps 中的所有属性转化为 ref（即响应性数据），便于 watch 进行监听



#### script setup 可以进一步简化组件代码

- 给 `script` 标签添加 `setup` 属性
- 可以直接当做在 `setup()` 函数中编写代码
- 不用手动 `return`返回变量，可以直接在模板中使用
- `import` 导入的变量也可以直接在模板中使用
- 定义 `props` 使用 `defineProps(`) ，定义事件使用 `defineEmits()`，编译器宏，无需导入
- 访问 `context`中的属性，可以分别使用 useAttrs() 和 useSlots()
- `template`模板中的代码保持不变

## Vue-Router

#### 路由组件不刷新的问题

- Vue-router 对同组件的路由跳转，是不会创建或者销毁组件，而是直接复用组件实例，这样就不会触发声明周期函数。
- 生命周期函数只在组件创建或销毁时执行一次

##### 解决方法：使用 watch 监听路由参数的变化，强制组件刷新

```js
watch: {
  // 注意此处直 接使用 $route.params 是由于watch 内的 this 本就指向 vue 实例
    "$route.params": {
      handler(params, oldParams) {
        this.blogPost = getBlogPostById(params.postId);
      },
       // watch 是懒加载的，需要设置 immediate: true 让其在组件加载的时候先执行一次 
      immediate: true,
    },
  },
```

#### 	匹配 404 写法

`/:notFound(.*)*` 

#### $router.push()  和 $router.replace() 的区别

$router.push() 是向历史记录中**追加新纪录**， $router.replace() 是**替换掉历史记录的最新记录**



#### 路由导航守卫

##### 	全局导航守卫（适合权限验证）

- ​	`router.beforeEach((to,from) => { })`

​	该钩子会在导航刚触发、组件还没有加载、且导航发生在实际跳转前

- ​	`router.beforeResolve((to,from) => { })`

​	该钩子会在导航守卫执行完毕、且组件加载完毕、组件中的导航守卫执行完毕之后、且导航实际跳转前执行

- ​	 `router.afterEach((to,from) => { })`

​	该钩子会在导航确认并实际跳转之后执行, 此时可以修改DOM，例如页面标题，可以在此处埋点发送统计数据 

##### 路由导航守卫

- 

  ```js
  {
    path:"blogs",
    component:BlogDetails,
     beforeEnter(to,from) {
      console.log(to)
    }
  }
  ```

  会在进入对应的路由时，页面实际跳转前执行，**但是**如果是按动态参数设置的 URL ，在他们之间进行跳转时，由于是组件复用，路由没有实际发生跳转，所以 `beforeEnter` 不会重新执行

##### 组件导航守卫

- `beforeRouteEnter((to,from,next) => {})`  导航跳转时，组件创建前执行，此时无法访问组件实例（ 使用动态参数渲染同一个组件不会重新执行）
- `beforeRouteUpdate((to,from) => {})` 导航跳转时，且复用组件时，此时可以访问组件实例（路由动态参数变化，组件复用的时候会执行）
- `beforeRouteLeave((to,from,next) => {})`  导航跳转时，组件销毁前执行（可以用于提示用户是否确认离开该页面）

##### 导航守卫执行顺序

1. 先执行组件中的 `beforeRouteLeave` 导航守卫（如果有的话）

2. 执行全局的 `beforeEach()` 导航守卫

3. 在复用的组件中执行 `beforeRouteUpdate()`导航守卫

4. 执行路由对象的 `beforeEnter`导航守卫

5. 解析异步的导航组件

6. 执行组件中的 `beforeRouterEnter()` 导航守卫

7. 执行全局的 `beforeResolve()`导航守卫

8. 确认导航，发生页面跳转  

9. 执行全局的 `afterEach()`导航守卫

10. 更新DOM

11. 执行组件中的 `beforeRouterEnter`中的 `next()` 回调函数中的回调函数

    

##### 页面滚动行为的控制

 给 `createRouter`传递一个  `scrollBehavior`配置

```js
const router = createRouter({
  history: createWebHistory(),
  routes,
  scrollBehavior(to, from, savedPosition) {
    // return { top: 0 };
    // return {
    //   top: 200,
    // 控制滚动条的平滑移动
    //   behavior: "smooth", 
    // };
    // 延迟
    // return new Promise((resolve) => {
    //   setTimeout(() => resolve({ top: 200, behavior: "smooth" }), 1000);
    // });
    // return {
    //   el: "#app",
    //   top: -60,
    // };
    console.log(savedPosition);
    if (savedPosition) {
      return savedPosition;
    } else {
      return { top: 0 };
    }
  },
});
```



##### 给组件传递 `Props`

实质是将动态参数以 Props 的形式传递给组件使用，避免组件与路由的高耦合度，提高组件的复用性

为路由对象配置 Props 属性

```js
{
    path: "/:postId",
    component: BlogPostPage,
    // props: true,
    // props: { postId: 5 },
    props: (route) => {
      // 这里可以做较为复杂的业务逻辑操作
      console.log(route);
      return { postId: Number(route.params.postId) };
    },
  },
```

 在组件中可以直接通过 props 访问

```js
export default {
  props: ["postId"],
  data() {
    return { blogPost: {} };
  },
  created() {
    // this.blogPost = getBlogPostById(this.$route.params.postId);
    console.log(this.postId);
    console.log(typeof this.postId); // 函数形式，可以改变 postId 的类型
    this.blogPost = getBlogPostById(this.postId);
  },
};
```



## Vuex

核心流程：组件 => mutaions =>  store => 所有使用目标属性的组件刷新

使用Vuex

```js
import { createStore } from 'vuex'
const store = createStore({
	state() {
		return {
			color:[100,100,100]
		}
	},
	mutations:{
		randomColor(state) {
			state.color = [Math.floor(Math.random()*255),Math.floor(Math.random()*255),Math.floor(Math.random()*255)]
		}
	}
})
// 注意要在组件挂载前使用 store，不然无法访问
app.use(store)
app.mount("#app");
```



##### 定义和访问 `state`

```js
import { mapState } from 'vuex'
 computed:{
    ...mapState({
      // 函数形式的访问可以增加逻辑操作
      num: (state) => state.num
    })
  },
```

```js
computed:{
    ...mapState({
      // 将全局的 num  参数映射到组件的 计算属性 num 参数上
      num: 'num'
    })
  },
```

```js
computed:{
  	// 直接同名映射
    ...mapState(['num','string'])
  },
```



##### 定义和触发 `Mutations`

可以通过 `this.$store.commit('changFn')` 直接触发

如果需要传递参数 `this.$store.commit({type:'changFn',ele:argment)`

```js
const store = createStore({
  state() {
    return {
      num: 1,
      arr: [1, 2, 3],
      user: {
        id: 1,
        name: "John",
        age: 25,
      },
    };
  },
  mutations: {
    // increment(state) {
    //   state.num++;
    // },
    
    // 配置第二个参数 payload ，用于接收触发 pushToArr 时传递过来的参数
    pushToArr(state, payload) {
      state.arr.push(payload.ele);
    },
    changeUserName(state, payload) {
      state.user.name = payload.name;
    },
  },
});
```



```js
import { mapState, mapMutations } from "vuex";

methods:{
 	// 将 mutations 中的 increment 方法直接映射到 methods 中 
	...mapMutations(['increment'])
}
```



##### 定义和访问 `getters`

```js
const store = createStore({
  state() {
    return {
      users: [
        {
          id: 1,
          name: "John",
          age: 25,
        },
        {
          id: 2,
          name: "Jane",
          age: 26,
        },
        {
          id: 3,
          name: "Jack",
          age: 27,
        },
        {
          id: 4,
          name: "Jill",
          age: 28,
        },
      ],
    };
  },
  getters: {
    usersOlderThan26(state) {
      return state.users.filter((user) => user.age > 26);
    },
    // 在numberOfUsersOlderThan26 继续使用其他 getters 属性
    numberOfUsersOlderThan26(state, getters) {
      return getters.usersOlderThan26.length;
    },
    // 使用时传入参数
    usersOlderThan(state) {
      // 这种情况不会缓存结果
      return (age) => state.users.filter((user) => user.age > age);
    },
  },
})
```

**访问 getters**

```js
import { mapGetters } from "vuex";
// 依旧可以将其映射到 computed 中
computed: {
  ...mapGetters([
    "usersOlderThan26",
    "numberOfUsersOlderThan26",
    "usersOlderThan",
  ]),
}
```



##### 定义和访问 Actions

```js
const store = createStore({
  state() {
    return {
      users: [],
      loading: false,
    };
  },
  mutations: {
    loadUsers(state, payload) {
      state.users = payload.users;
    },
    setLoading(state, loading) {
      state.loading = loading;
    },
  },
  actions: {
    // 传入 context 以访问 commit 触发 mutations
    // fetchUsers(context) {
    //   setTimeout(() => {
    //     context.commit("loadUsers", { users });
    //   }, 2000);
    // },
    // 需要传入参数的情况
    fetchUsers({ commit }, payload) {
      commit("setLoading", true);
      setTimeout(() => {
        commit("loadUsers", { users: users.slice(0, payload.limit) });
        commit("setLoading", false);
      }, 2000);
    },
  },
});
```

**在组件中触发** `actions`

可以通过`this.$store.dispatch('fetchUsers')`

```js
import { mapState, mapActions } from "vuex";
created() {
    // this.$store.dispatch("fetchUsers");
    // this.fetchUsers();
  	// 触发 actions 并传入参数
    this.fetchUsers({ limit: 3 });
  },

  computed: mapState(["users", "loading"]),
	// 将 actions 中的方法映射到 methods
  methods: mapActions(["fetchUsers"]),
```

> `mutations` 中的代码必须是同步的，这是为了方便追踪状态，在调试的时候清楚的知道状态时如何变化的，变化的路径是什么
>
> 不能使用 `v-model`直接绑定 `state`中的值，需要使用计算属性



##### `modules` 

`actions` 可以触发所有模块中的 `actions`、`mutations` ，`getters`可以访问所有模块中的 `getters`, `mutations`只能修改自身模块的 `state`



##### `modules`命名空间，防止各模块出现重复的属性名

#####  

```js
export const blogs  = {
  namespaced:true
}

commit("blogs/ad")
```

