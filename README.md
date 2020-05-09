# 起因

引入vue tag 这段代码在ant-design-vue 1.4.9版本生效，1.5.4 版本不生效 nextTick不生效

~~~
<template>
  <div>
    <template v-for="(tag, index) in tags">
      <a-tooltip v-if="tag.length > 20" :key="tag" :title="tag">
        <a-tag :key="tag" :closable="index !== 0" @close="() => handleClose(tag)">
          {{ `${tag.slice(0, 20)}...` }}
        </a-tag>
      </a-tooltip>
      <a-tag v-else :key="tag" :closable="index !== 0" @close="() => handleClose(tag)">
        {{ tag }}
      </a-tag>
    </template>
    <a-input
      v-if="inputVisible"
      ref="input"
      type="text"
      size="small"
      :style="{ width: '78px' }"
      :value="inputValue"
      @change="handleInputChange"
      @blur="handleInputConfirm"
      @keyup.enter="handleInputConfirm"
    />
    <a-tag v-else style="background: #fff; borderStyle: dashed;" @click="showInput">
      <a-icon type="plus" /> New Tag
    </a-tag>
  </div>
</template>
<script>
export default {
  data() {
    return {
      tags: ['Unremovable', 'Tag 2', 'Tag 3Tag 3Tag 3Tag 3Tag 3Tag 3Tag 3'],
      inputVisible: false,
      inputValue: '',
    };
  },
  methods: {
    handleClose(removedTag) {
      const tags = this.tags.filter(tag => tag !== removedTag);
      console.log(tags);
      this.tags = tags;
    },

    showInput() {
      this.inputVisible = true;
      this.$nextTick(function() {
        this.$refs.input.focus();
      });
    },

    handleInputChange(e) {
      this.inputValue = e.target.value;
    },

    handleInputConfirm() {
      const inputValue = this.inputValue;
      let tags = this.tags;
      if (inputValue && tags.indexOf(inputValue) === -1) {
        tags = [...tags, inputValue];
      }
      console.log(tags);
      Object.assign(this, {
        tags,
        inputVisible: false,
        inputValue: '',
      });
    },
  },
};
</script>
~~~

上诉代码在一个Dialog执行，在ant-design-vue 1.4.9版本生效，1.5.4版本，
~~~
showInput（）{
          this.$nextTick(function() {
        this.$refs.input.focus();
      })；
}
~~~
不生效。nextTick并不是在页面重绘之后执行，也就是Input被创建，有了引用之后，才执行，此时获取的this.$refs.input仍然为undefined，会报错。
并且，该代码运行在https://blog.codepen.io/documentation/preview-template/ 1.5.4版本不存在问题，当然运行在codepen上的代码没有dialog包裹，不知道与这个原因有关否

# 解决方案

>让nextTick的返回结果为一个promise，在promise结束之后，再去调用 this.$refs.input.focus();
~~~
this.$nextTick().then( ()=> {
     console.log(this.$refs.inputRef)
     this.inputRef.focus();
});
~~~
因为this.$nextTick(cb?:Fucntion)不传递时，就返回一个Promise

可看this.$nextTick源码

~~~
//将nextTick定义到Vue原型链上代码位于src/core/instance/render.js
Vue.prototype.$nextTick = function (fn: Function) {
return nextTick(fn, this)
 }
~~~

~~~
//传入回调函数cb,如果cb为undefined并且该运行环境支持Promise ,就返回一个Promise
export function nextTick (cb?: Function, ctx?: Object) { // next-tick.js 
  //是否完成的函数
  let _resolve
  callbacks.push(() =&gt; {
    //如果cb不为undefined
    if (cb) {
      try {
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      _resolve(ctx)
    }
  })

  if (!pending) {
    pending = true
    timerFunc()
  }

  if (!cb &amp;&amp; typeof Promise !== 'undefined') {
    return new Promise(resolve =&gt; {
      _resolve = resolve
    })
  }
}
~~~

或者
~~~
            setTimeout(() => {
                this.input.focus();
            })
~~~

可知传入的回调函数，并非如同官网所说
>在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的DOM
猜测，没有数据的改变，只是v-if来控制显示，有可能正在DOM更新循环前执行的cb正在执行

# 宏任务 与 微任务

## 第一个例子
~~~
                console.log('script start');

                setTimeout(function() {
                    console.log('setTimeout');
                }, 0);

                Promise.resolve().then(function() {
                    console.log('promise1');
                }).then(function() {
                    console.log('promise2');
                });

                console.log('script end');
~~~
打印：

script start 

script end

promise1

promise2

setTimeout

### 原理
 

### 为什么Promise比setTimeout先执行
```flow
st=>start: 用户登陆
op=>operation: 登陆操作
cond=>condition: 登陆成功 Yes or No?
e=>end: 进入后台

st->op->cond
cond(yes)->e
cond(no)->op
```
# 起因

引入vue tag 这段代码在ant-design-vue 1.4.9版本生效，1.5.4 版本不生效 nextTick不生效

~~~
<template>
  <div>
    <template v-for="(tag, index) in tags">
      <a-tooltip v-if="tag.length > 20" :key="tag" :title="tag">
        <a-tag :key="tag" :closable="index !== 0" @close="() => handleClose(tag)">
          {{ `${tag.slice(0, 20)}...` }}
        </a-tag>
      </a-tooltip>
      <a-tag v-else :key="tag" :closable="index !== 0" @close="() => handleClose(tag)">
        {{ tag }}
      </a-tag>
    </template>
    <a-input
      v-if="inputVisible"
      ref="input"
      type="text"
      size="small"
      :style="{ width: '78px' }"
      :value="inputValue"
      @change="handleInputChange"
      @blur="handleInputConfirm"
      @keyup.enter="handleInputConfirm"
    />
    <a-tag v-else style="background: #fff; borderStyle: dashed;" @click="showInput">
      <a-icon type="plus" /> New Tag
    </a-tag>
  </div>
</template>
<script>
export default {
  data() {
    return {
      tags: ['Unremovable', 'Tag 2', 'Tag 3Tag 3Tag 3Tag 3Tag 3Tag 3Tag 3'],
      inputVisible: false,
      inputValue: '',
    };
  },
  methods: {
    handleClose(removedTag) {
      const tags = this.tags.filter(tag => tag !== removedTag);
      console.log(tags);
      this.tags = tags;
    },

    showInput() {
      this.inputVisible = true;
      this.$nextTick(function() {
        this.$refs.input.focus();
      });
    },

    handleInputChange(e) {
      this.inputValue = e.target.value;
    },

    handleInputConfirm() {
      const inputValue = this.inputValue;
      let tags = this.tags;
      if (inputValue && tags.indexOf(inputValue) === -1) {
        tags = [...tags, inputValue];
      }
      console.log(tags);
      Object.assign(this, {
        tags,
        inputVisible: false,
        inputValue: '',
      });
    },
  },
};
</script>
~~~

上诉代码在一个Dialog执行，在ant-design-vue 1.4.9版本生效，1.5.4版本，
~~~
showInput（）{
          this.$nextTick(function() {
        this.$refs.input.focus();
      })；
}
~~~
不生效。nextTick并不是在页面重绘之后执行，也就是Input被创建，有了引用之后，才执行，此时获取的this.$refs.input仍然为undefined，会报错。
并且，该代码运行在https://blog.codepen.io/documentation/preview-template/ 1.5.4版本不存在问题，当然运行在codepen上的代码没有dialog包裹，不知道与这个原因有关否

# 解决方案

>让nextTick的返回结果为一个promise，在promise结束之后，再去调用 this.$refs.input.focus();
~~~
this.$nextTick().then( ()=> {
     console.log(this.$refs.inputRef)
     this.inputRef.focus();
});
~~~
因为this.$nextTick(cb?:Fucntion)不传递时，就返回一个Promise

可看this.$nextTick源码

~~~
//将nextTick定义到Vue原型链上代码位于src/core/instance/render.js
Vue.prototype.$nextTick = function (fn: Function) {
return nextTick(fn, this)
 }
~~~

~~~
//传入回调函数cb,如果cb为undefined并且该运行环境支持Promise ,就返回一个Promise
export function nextTick (cb?: Function, ctx?: Object) { // next-tick.js 
  //是否完成的函数
  let _resolve
  callbacks.push(() =&gt; {
    //如果cb不为undefined
    if (cb) {
      try {
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      _resolve(ctx)
    }
  })

  if (!pending) {
    pending = true
    timerFunc()
  }

  if (!cb &amp;&amp; typeof Promise !== 'undefined') {
    return new Promise(resolve =&gt; {
      _resolve = resolve
    })
  }
}
~~~

或者
~~~
            setTimeout(() => {
                this.input.focus();
            })
~~~

可知传入的回调函数，并非如同官网所说
>在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的DOM
猜测，没有数据的改变，只是v-if来控制显示，有可能正在DOM更新循环前执行的cb正在执行

# 宏任务 与 微任务

## 第一个例子
~~~
                console.log('script start');

                setTimeout(function() {
                    console.log('setTimeout');
                }, 0);

                Promise.resolve().then(function() {
                    console.log('promise1');
                }).then(function() {
                    console.log('promise2');
                });

                console.log('script end');
~~~
打印：

script start 

script end

promise1

promise2

setTimeout

### 原理
 

### 为什么Promise比setTimeout先执行
```flow
st=>start: 用户登陆
op=>operation: 登陆操作
cond=>condition: 登陆成功 Yes or No?
e=>end: 进入后台

st->op->cond
cond(yes)->e
cond(no)->op
```
