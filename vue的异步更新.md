# <p align="center">个人关于vue的响应式的理解笔记</p>


#### 受现代 ``JavaScript`` 的限制 (而且 ``Object.observe`` 也已经被废弃)，<font color="pink">Vue不能检测到对象属性的添加或删除</font>。由于<font color="green">Vue会在初始化实例时对 属性 执行 ``getter/setter`` 转化过程</font>，所以<font color="green">属性必须在 ``data对象上存在`` 才能让 Vue 转换它，这样才能让它是响应式的</font> 。  
### ( <font color="pink">只有 ``在data对象上存在的属性`` 才能让 vue 转换，这样它才是 ``响应式的`` )
###  ( Vue不允许动态添加根级响应式属性 , 初始化实例前 ``声明根级响应式属性，哪怕只是一个空值 `` )</font>
              var vm = new Vue({
                data:{
                  a:1,
                  message: ''         // 声明 message 为空值字符串
                },
                template: '<div>{{ message }}</div>'
              })

              vm.message = 'hello'    // 设置 'message'
              
              // vm.a 是响应的

              vm.b = 2          // vm.b 是非响应的,因为该属性不存在于data对象上  
              
### <font color="pink">Vue 不允许在已经创建的实例上动态添加新的根级响应式属性</font> ``(root-level reactive property)`` ,然而可以使用 
### ``Vue.set(object, key, value)方法`` 将响应属性添加到嵌套的对象上
              Vue.set(vm.someObject, 'b', 2)  
      
### 也可用 ``vm.$set方法`` ，这是 `` 全局Vue.set方法`` 的别名  
              this.$set(this.someObject,'b',2)  
      
### <font color="green">向一个已有对象添加多个属性，可使用 ``Object.assign()`` 或 ``_.extend()方法`` 来添加属性</font>。<font color='pink'>该方法添加到对象上的新属性不会触发更新,在这种情况下可以创建一个新的对象，让它包含原对象的属性和新的属性</font>  
      
              // 代替 Object.assign(this.someObject, { a: 1, b: 2 })  
              this.someObject = Object.assign({}, this.someObject, { a: 1, b: 2 })  


#### <font color="green">为了在数据变化之后等待 Vue 完成更新 DOM ，可以在 ``数据变化之后`` 立即使用 `` Vue.nextTick(callback) `` , 这样 ``回调函数`` 在 ``DOM更新完成后`` 就会调用。</font>  
#### <font color="pink">( 在组件内使用 ``vm.$nextTick(callback)实例方法`` 更好，因为它 ``不需要 全局Vue ，且回调函数中的 this 将自动绑定 到当前的Vue实例上`` )</font>
#### ( ``$nextTick() 返回一个 Promise对象`` ，所以可以使用新的 ES2016 async/await 语法完成相同的事情 ) 
### <font color="pink">( ***不能在选项属性或回调上使用箭头函数 , ``箭头函数是和父级上下文绑定在一起的`` ， ``this 指向的不是如你所预期的 Vue 实例``</font>*** ，如 
            created: () => console.log(this.a)   
             或    
            vm.$watch('a', newValue => this.myMethod())          
##### 容易导致 ``Uncaught TypeError: Cannot read property of undefined 或 Uncaught TypeError: this.myMethod is not a function 之类的错误``

</br>

### <font color="pink">( 所有的方法都应该在methods里定义，然后在created或者mounted里 使用this调用方法，用这种方式实现初始化 )</font>
              
            <!DOCTYPE html>
            <html lang="en">
            <head>
              <meta charset="UTF-8">
              <meta name="viewport" content="width=device-width, initial-scale=1.0">
              <meta http-equiv="X-UA-Compatible" content="ie=edge">
              <title>vue异步更新</title>
              <script src="https://cdn.jsdelivr.net/npm/vue@2.6.8/dist/vue.js"></script>
            </head>
            <body>
              <div id="app">
                <h1>{{message}}</h1>
                <example></example>
              </div>
            </body>  
            
            <script>
              Vue.component('example', {
                  template: '<span>{{msn}}</span>',
                  data: () => {
                    return{
                        msn: '更新了？'
                    }
                  },
                  mounted() {
                      this.updateMsn();
                  },
                  methods: {
                      updateMsn: async function() {
                          console.log(1)
                          this.msn = '更新好了'
                          console.log(this.$el.textContent)	// data数据未刷新
                          await this.$nextTick(() => {
                              console.log(this.$el.textContent)	// data数据刷新了
                          })
                      }
                  }	
              }),
              new Vue({
                el: '#app',
                data: {
                  message: 'Vue的异步更新'
                }

              })
            </script>
            </html> 
      