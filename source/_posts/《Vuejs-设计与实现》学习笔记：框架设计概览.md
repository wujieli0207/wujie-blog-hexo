---
title: 《Vuejs 设计与实现》学习笔记：框架设计概览
date: 2022-03-23 12:42:32
categories: 前端
tags: vue
---

### 第 1 章：权衡的艺术

- 命令式和声明式
  - 命令式：关注目的实现的**过程**，比如 JQuery、直接操作 Dom
  - 声明式：关注目的实现的**结果**，不在乎如何实现
    ```vue
    <div @click="() => alert('hello')">hello</div>
    ```
  - 声明式框架内部封装的是命令式代码
  - **声明式的性能 <= 命令式**
    - 命令式直接操作，性能 = B(直接修改)
    - 声明式需要找出区别，再通过命令操作，性能 = A(找出差异) + B(直接修改)
  - 声明式的可维护性 > 命令式
  - 设计声明式框架，重点在于：**保持可维护性的同时，尽可能提升性能**
- 虚拟 DOM 性能比较

  - 三种常用创建页面方式：innerHTML、虚拟 DOM、原生操作

    |          | innerHTML | 虚拟 DOM | 原生 |
    | -------- | --------- | -------- | ---- |
    | 性能     | 差        | 一般     | 最好 |
    | 可维护性 | 一般      | 最好     | 差   |
    | 心智负担 | 一般      | 最好     | 差   |

  - 新建页面时，innerHTML 和虚拟 DOM 性能差距不大
  - innerHTML 性能差的原因：在更新页面时，需要销毁所有旧 DOM，重新创建新 DOM，**页面越大，性能越差**
  - 虚拟 DOM 只更新必要的 DOM，和页面大小无关

- 运行时和编译时
  - 运行时：代码直接运行，缺点是不能分析提供的代码内容，缺少性能
  - 编译时：将代码进行分析转换为可执行的代码，缺点是灵活性较差
  - Vue.js 是运行时 + 编译时架构，**保证灵活性的基础上，通过分析，尽可能提升性能**

### 第 2 章：框架设计的核心要素

- **提升用户的开发体验**：比如明显的告警错误提示
  - 快速定位问题
  - 节省时间
- **控制代码体积**
  - 框架层面支持：比如区分 dev 和 prod 环境，dev 环境执行的错误提示等代码在 prod 环境可以剔除，**不会增加生产代码体积**
  - Tree-Shaking：排除永远不会执行的代码
    - 必须是 ESModule 才支持 Tree-Shaking
    - 通过静态扫描排除不必要的代码
    - 如果代码产生副作用（对外部产生影响），则代码不会被剔除。一般只有顶级调用的函数才可能出现副作用
    - 可以通过增加 `/*#__PURE__*/` 显示声明代码可以被 Tree-Shaking 剔除
- **支持输出多种构建产物**
  - 根据打包和运行环境不同，可是通过 IIFE 格式直接引用，可以输出支持 webpack / rollup 的 esm-bundler 文件，也可输出支持浏览器直接运行的 esm-browser 文件，还可以支持 node.js 运行的文件
  - 以上输出可以使用不同参数配置
- **特性开关**
  - 框架的各种特性可以通过不同的开关来处理
  - 特性开关的好处：
    - 更好的使用 Tree-Shaking
    - 灵活性：
      1. 通过特性开关增加新特性而不用担心打包文件体积
      2. 框架升级时可以通过特性开关保留需要的功能，利于平滑升级
- **错误处理**
  - 错误处理决定代码的健壮性
  - 内部使用一个统一的方法 `callWithErrorHandling` 来捕获错误
  - 通过 `registerErrorHandle` 函数可以注册错误处理程序，处理 `callWithErrorHandling` 捕获到的错误
- **良好的 TS 类型支持**
  - 使用 TS 编写代码不等于良好的 TS 类型支持，良好的类型支持需要能够更直观的展示类型
  - Vue.js 源码的 `runtime-core/src.apiDefineComponent.ts` 主要为 TS 类型支持服务

### 第 3 章：Vue.js 3 的设计思路

- 声明式描述 UI
  - 使用 HTML 描述页面涉及如下四个方面
    - DOM 元素
    - 属性：id、class ...
    - 时间：click
    - 元素的层级结构
  - 针对以上四个方面，Vue 提供的解决方案
    - 使用与 HTML 标签一致的方式描述 DOM 元素
    - 使用与 HTML 标签一致的方式描述基础属性，使用 `:` 或 `v-bind` 描述动态属性
    - 使用 `@` 或 `v-on` 描述事件
    - 使用和 HTML 一样的方式描述层级结构
  - 除了使用模板的方式描述页面，还可以使用 JS 对象的方式描述（即虚拟 DOM）UI，再通过渲染器渲染页面
  - h 函数是一个辅助创建虚拟 DOM 的工具函数
- 渲染器基础

  - 渲染器作用：把虚拟 DOM 渲染为真实 DOM
  - 渲染器的实现思路
    - 接受两个参数：虚拟 DOM、挂载点（真实 DOM，用于放置被渲染的虚拟 DOM）
    - 创建元素：通过 `vnode.tag` 创建 DOM 元素
    - 处理属性和事件：如果属性以 on 开通，说明是事件，截取 on 之后的函数内容添加到 `addEventListener` 函数
    - 处理 children：如果是一个数组，则递归调用 render 继续渲染，如果是一个 string，则通过 `createTextNode` 函数创建一个文本节点
  - **渲染器的核心在于准确找到变更节点并变更内容**
  - 创建渲染器参考函数

    ```js
    function renderer(vnode, container) {
      // 使用 vnode.tag 作为标签名创建 DOM 元素
      const el = document.createElement(vnode.tag);
      // 遍历 vnode.props，将属性、事件添加到 DOM 元素
      for (const key in vnode.props) {
        if (/^in/.test(key)) {
          el.addEventListener(key.substr(2).toLowerCase(), vnode.props[key]);
        }
      }

      // 处理 children
      if (typeof vnode.children === "string") {
        el.appendChild(document.createTextNode(vnode.children));
      } else if (Array.isArray(vnode.children)) {
        vnode.children.forEach((child) => {
          renderer(child, el);
        });
      }

      // 将元素添加到挂载点下
      container.appendChild(el);
    }
    ```

- 组件的本质

  - **组件就是一组 DOM 元素的封装**，可以由函数组成，也可以由对象组成
  - 处理渲染组件的函数是 `mountComponent`

    - 实现步骤

      1. 获取要渲染的内容（虚拟 DOM）
      2. 递归调用 render 渲染

    - 函数式组件渲染
      ```js
      function mountComponent(vnode, container) {
        const subtree = vnode.tag();
        renderer(subtree, container);
      }
      ```
    - 对象式组件渲染
      ```js
      function mountComponent(vnode, container) {
        const subtree = vnode.tag.render(); // 调用对象的 render 函数获取虚拟 DOM
        renderer(subtree, container);
      }
      ```

- 编译器的作用：将模板编译为渲染函数
- 编译器将模板编译为渲染函数，渲染函数将虚拟 DOM 转化为真实 DOM，两者相互依赖，可以借助对方提升整体性能（比如编译器告诉渲染器哪些元素是静态，更新的时候渲染函数就可以只针对动态内容更新，提升效率）
