# Vue3.x composition-api介绍

## 生命周期

> 1，新增了两个hook，onRenderTriggered、onRenderTracked   
> 2，原有的hook全部保留而且特性一致   
> 3，提供新版的函数组合的方式使用，2.x中选项类型的使用方式依旧保留

```
import {
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted,
  onRenderTriggered,
  onRenderTracked
} form 'vue'

export default {
  setup () {
    onBeforeMount(() => {
      // do something...
    })
  }
}
```

## 组件实例
> 1，setup 接收props、context两个参数   
> 2，在setup中无法使用this

```
import { getCurrentInstance } from 'vue'

export default {
  // 属性定义方式和上版本一致 
  props: {
    value: String
  }
  setup (props, context) {
    //  获取当前组件实例
    const instance = getCurrentInstance()
  }
}
```

## 响应式

### ref
[ref源码地址](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/ref.ts) 
> 3.x版本新增的一个api、这个api也是新版api中理解跨度比较大的点

在2.x版本中我们使用响应式数据直接 通过在data返回就行了，然后在使用到的地方直接`this.xxx`
```
export default {
  data () {
    return {
      say: 'hello world'
    }
  },
  methods: {
    print () {
      console.log(this.say)
    }
  }
}
```
但是3.x版本在`setup`中是无法使用`this`的，同时还有 一个问题3.x中的响应式是借助`proxy`来实现，所以对于javascript里面的基础类型的监听就达不到目的
```
export default {
  setup () {
    let say = 'hello world'

    const openmouse = () => {
      say = '你好！'
    }
    
    return {
      say,
      openmouse
    }
  }
}
```
如果我们执行了`openmouse`因为say不是响应式的页面没有任务变化
```
import { ref } from 'vue'

export default {
  setup () {
    // ref会返回一个处理过的包装对象
    const say = ref('hello world')
    
    const openmouse = () => {
      // 赋值时
      say.value = '你好！'
    }

    // 获取值
    // 在template模版中使用不需要.value，直接用使用say就行vue内部有识别处理
    console.log(say.value)
    
    return {
      say,
      openmouse
    }
  }
}
```   

**ref可以接收任何类型的值、如果是一个Object默认会deep处理**   
    
  
##### ref相关更多api
```
import {
  ref,
  shallowRef,
  isRef,
  toRef,
  toRefs,
} from 'vue'
```
* ref：使一个js类型数据具有响应性
* shallowRef：ref如果接收的参数是一个Object会处理这个Object每一个key，shallowRef只处理这个Object本身

```
import {
  ref,
  shallowRef,
  isRef,
  toRefs,
} from 'vue'

export default{
  setup () {
    const personRef =  ref({
      name: 'bad gay'
    })
    const personShallowRef = shallowRef({
      name: 'bad gay'
    })
    
    const click1 = () => {
      // 会触发更新
      personRef.value.name = 'good boy'
      
    }
    
    const click2 = () => {
      // 不会触发更新
      personShallowRef.value.name = 'good boy'
    }
    
    const click3 =  () => {
      // 会触发更新
      personShallowRef.value = {
        name: 'good boy'
      }
    }
    
    return {
      personRef,
      click1,
      personShallowRef,
      click2,
      click3
    }
  }
}
```
* isRef：判断一个值是否是ref
* toRefs：接收一个Object参数，一般用来使reactive类型的值保持引用

### reactive

[reactive源码地址](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/reactive.ts)

> 1，字面意思可以看出是用来使数据具有响应性的 ，接收的参数一定是个Object  
> 2，和ref有相同的作用，又有一定的区别

```
import { reactive } from 'vue'

export default {
  setup () {
    const state = reactive({
      type: 'RMB',
      count: 0
    })
    
    const inc = () => {
      state.count++
    }
    
    return {
      state,
      inc
    }
  }
}
```
#### reactive和ref的区别

前面说了ref使用来解决`proxy`无法监听js基本类型的问题，所以ref接收参数是string、number、boolean、null、undefine，当然Object一样可以处理;    
如果这个数据是Object类型的，建议使用reactive
```
<template>
  <div>{{ numberRef }}</div>
  <div>{{ stringRef }}</div>
  <div>{{ booleanRef }}</div>
  <div>{{ nullRef }}</div>
  <div>{{ undefineRef }}</div>
  <div>{{ objectRef.count }}</div>
  <div>{{ objectValue.count }}</div>
</template>
<script>
  import { ref, reactive } from 'vue'

  export default {
    setup () {
      // js基本类型 
      const numberRef = ref(0)
      const stringRef = ref('')
      const booleanRef = ref(false)
      const nullRef = ref(null)
      const undefineRef = ref(undefined)
      
      //  js引用类型
      const objectRef = ref({
        count: 0
      })
      const objectValue = reactive({
        count: 0
      })
	  
      // 引用类型时ref(.value)和reactive的使用区别
      objectRef.value.count =  2
	  console.log(objectRef.value.count)
	  
      objectValue.count = 2
      console.log(objectValue.count)
      
      return {
        numberRef,
        stringRef,
        booleanRef,
        nullRef,
        undefineRef,
        objectRef,
        objectValue
      }
    }
  }
</script>
```
#### reactive使用展开符(...)
```
<template>
  <div>{{ count }}</div>
  <button @click="baofu">暴富</button>
</template>
<script>
  import { reactive } from 'vue'

  export default {
    setup () {
      const state = reactive({
        type: 'RMB',
        count: 0
      })
      
      // 因为在return时state被展开了点击按钮后页面不会更新
      const baofu = () => {
        state.count  = Math.max()
      }
      
      return {
        // reactive 使用展开符，type和count没有响应式特性了
        ...state,
        baofu
      }
    }
  }
</script>
```
reactive配合toRefs使用展开符
```
<template>
  <div>{{ count }}</div>
  <button @click="baofu">暴富</button>
</template>
<script>
  import { reactive, toRefs } from 'vue'

  export default {
    setup () {
      const state = reactive({
        type: 'RMB',
        count: 0
      })
      
      // 点击后页面会更新
      const baofu = () => {
        state.count  = Math.max()
      }
      
      return {
        // 用toRefs包裹一层，type和count依旧有响应式特性了
        ...toRefs(state),
        baofu
      }
    }
  }
</script>
```

## computed

[computed源码地址](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/computed.ts)

> 1，和生命周期一样提供函数组合的方式保留了选项类型的使用方式   
> 2，主要是api的变化，内部实现和特性几乎和2.x一样 

```
<template>
  <div>{{ caifu }}+{{ state.count }}</div>
  <button @click="baofu">暴富</button>
</template>
<script>
  import { reactive, computed } from 'vue'

  export default {
    setup () {
      const state = reactive({
        type: 'RMB',
        count: 0
      })
      
      const caifu = computed(() => {
        if (state.count < 1) {
          return '贫穷'
        }
        return '土豪'
      })
      
      const baofu = () => {
        state.count++
      }
      
      return {
        state,
        baofu,
        caifu
      }
    }
  }
</script>
```

## watch

[watch源码地址](https://github.com/vuejs/vue-next/blob/master/packages/runtime-core/src/apiWatch.ts)

> 1，和生命周期一样提供函数组合的方式保留了选项类型的使用方式   
> 2，增加了一个新的相关api——watchEffect   
> 3，和2.x 版本this.$watch使用方式相同，不过更强大

```
import { watchEffect } from 'vue'

export default {
  setup () {
    const say = ref('hello world')
    const state = reactive({
      type: 'RMB',
      count: 0
    })

    // 支持上个版本this.$watch所有写法
    const unWatch = watch(say, (newValue, oldValue) => {
      console.log('from watch', newValue)
    }, {
      immediate: true,
      deep: true,
      flush: true
    })

    watch(() => state.count, newValue => {
      console.log('有钱吗 ？', newValue)
    })
    // 支持数组
    watch([state.count, state.type], (newValueArr, oldValueArr) => {
      // newValueArr[0] = state.count
      // newValueArr[1] = state.type
      console.log('from watch mult', newValueArr)
    })

    return {
      say,
      state
    }
  }
}
```
#### watchEffect
> 会立即执行回掉，并将该执行过程中用到的所有响应式状态作为依赖进行追踪
```
import { watchEffect } from 'vue'

export default {
  setup () {
    const say = ref('hello world')
    const state = reactive({
      type: 'RMB',
      count: 0
    })

    watchEffect(() => {
      // 将该执行过程中用到的所有响应式状态作为依赖进行追踪
      console.log('当state.type和say有变化时会自动执行', state.type, say.value)
    })

    return {
      say,
      state
    }
  }
}
```
