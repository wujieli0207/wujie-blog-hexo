---
title: vue3个人常用通信方式总结
date: 2021-12-06 15:11:16
categories: 前端
tags: vue
---

## props

- Vue3 子组件可以通过 props 接受父组件的传值
- 在 setup() 中可以通过 `props.value` 来访问父组件传值
- 父组件传值举例

  ```html
  <template>
    <child-component :value="value" />
  </template>
  ```

- 子组件接受并使用父组件传递值
  <!--more-->

      ```html
      <template>
      	{{ fatherValue }}
      </template>
      <script lang="js">
          import { defineComponent, ref } from "vue";

          export default defineComponent ({
              name: "ChildComponent",
              props: {
                  value: {
                      type: String,
                      default: ""
                  }
              },
              setup( props ) {
                  const fatherValue = ref("");

                  fatherValue = props.value
                  console.log( fatherValue )

                  return {
                      fatherValue
                  }
              }
          })
      </script>
      ```

## refs

- 通过 refs ，父组件可以直接获取子组件实例，并向子组件传值
- 父组件举例

  ```html
  <template>
    <child ref="childRef" />
    <button @click="sendValue()">Change Child Value</button>
  </template>
  <script lang="js">
    import { defineComponent, ref } from "vue";

    export default defineComponent ({
        name: "ChildComponent",
        props: {
            value: {
                type: String,
                default: ""
            }
        },
        setup( props ) {
            const childRef = ref("");

            const sendValue = () => {
                console.log("childRef", childRef.value);

                // 调用子组件方法
                childRef.value.acceptValue("newValue");
            }

            return {
                fatherValue
            }
        }
    })
  </script>
  ```

- 子组件举例

  ```html
  <template> Son: {{ valueRef }} </template>
  <script lang="js">
    import { defineComponent, ref } from "vue";

    export default defineComponent ({
        name: "ChildComponent",
        props: {
            value: {
                type: String,
                default: ""
            }
        },
        setup( props ) {
            const valueRef = ref("");

            const acceptValue = (value) => {
            	valueRef.value = value;
            }

            return {
                valueRef,
                acceptValue
            }
        }
    })
  </script>
  ```

## emits

- 子组件可以通过 emit 暴露属性，父组件通过 emit 暴露属性操作子组件，比如：点击子组件触发了一个父组件大函数
- 父亲组件举例

  ```html
  <template> <child-component :add-child="addChildMethod()" </template>

  <script lang="js">
    import { defineComponent, ref } from "vue";

    export default defineComponent ({
        name: "FatherComponent",
        setup( props ) {
            const addChildMethod = () => {
                // mehtod
            }
            return {
                addChildMethod
            }
        }
    })
  </script>
  ```

- 子组件举例
  ```html
  <template>
    <div @click="$emit('addChild')">
      <!-- some -->
    </div>
  </template>
  ```
