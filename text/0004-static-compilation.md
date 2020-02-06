- 日期: 2020-02-06
- RFC PR:
- Issue:

# 总结

引入静态编译能力，在模板层做静态分析，优化生成更静态的小程序模板，以达到提升性能的目的。
静态编译与 React 运行时共同存在，结合使用，并非二选一方法。

# 基本例子

// remax.config.js
// 引入 templateCompiler 配置项，默认为 default 。static 表示开启静态编译模式。
{
templateCompiler: 'static' | 'default'
}

# 背景

出于对性能优化，模板大小控制（微信平台无法模板递归的限制）的考虑，以及小程序本身静态模板的特性限制，Remax 也应具备支持静态编译能力。

# 详细设计

设计的核心是，将模板中的节点静态化，并可与虚拟 dom 节点对应起来。

## Runtime

Runtime 层面不受影响，维持原样。

## 模板层

静态编译能力主要针对模板层做改造。

### 处理模板的步骤

1. 记录所有 JSX 片段，用于转换成小程序的模板
2. 将所有 JSX 表达式用 block 标签包裹，使之成为一个子节点（目的是保证这个节点可数，可与虚拟 dom 节点对应）
3. 任何无法确定的节点（React 组件节点，原生组件节点，无法判断来源的节点等）都会按照 JSX 表达式处理。
4. JSXText 特殊处理（JSX 中任何换行，空格都会算作 JSXText，因此需要预处理去掉不必要的片段）。
5. 给每个完整的 JSX 片段的最外层记录一个 template id 属性。
6. 根据每个 JSX 片段以及其对应的 template id 生成一个 小程序 template，以 template id 命名。
7. 渲染时，可以通过虚拟 dom 拿到 template id，从而知道调用哪个小程序模板。
8. 渲染时，无法确定 template id 的节点都被认为是无法静态化的节点（如上述提到的 JSX 表达式，React 组件，原生组件节点等），无法静态化的节点，采用现有的动态模板形式渲染。

## 例子

```jsx
import * as React from 'react';
import { View, View as CustomView } from 'remax/alipay';

const [str] = React.useState('string');
const ReactComp = () => <View>react component</View>

// jsx
<View>
  {str}
  <CustomView>literal</CustomView>
  <ReactComp />
  {React.cloneElement(<View />)}
  {[1, 2, 3].map(item => (
    <View>{item}</View>
  ))}
</View>

// 小程序模板
<template name="TPL_id_page">
  <view>
    <template is="TPL_DEFAULT" data="{{ node: node.children[0].children[0] }}" />
    <view>literal</view>
    <template is="TPL_DEFAULT" data="{{ node: node.children[0].children[2] }}" />
    <template is="TPL_DEFAULT" data="{{ node: node.children[0].children[3] }}" />
    <template is="TPL_DEFAULT" data="{{ node: node.children[0].children[4] }}" />
  </view>
</template>

<template name="TPL_id_react_comp">
  <view>react component</view>
</template>

<template name="TPL_id_clone_element">
  <view></view>
</template>

<template name="TPL_id_map">
  <view>
    <template is="TPL_DEFAULT" data="{{ node: node.children[0].children[0] }}" />
  </view>
</template>

<template name="TPL_DEFAULT">
  遍历节点：
    如果虚拟 dom 有模板 id，则调用模板
    如果没有，则采用动态模板的方式渲染
</template>

// 虚拟 dom
{
  root: {
    children: [{
      type: 'view',
      children: [{
        type: 'block',
        children: [{
          type: 'plain-text',
          text: 'string',
        }]
      }, {
        type: 'view',
        children: [{
          type: 'plain-text',
          text: 'literal'
        }]
      }, {
        type: 'block',
        children: [{
          type: 'view',
          children: [{
            type: 'plain-text',
            text: 'react component'
          }]
        }]
      }, {
        type: 'block',
        children: [{
          type: 'view',
        }]
      }, {
        type: 'block',
        children: [{
          type: 'view',
          children: [{
            type: 'plain-text',
            text: 1,
          }]
        }, {
          type: 'view',
          children: [{
            type: 'plain-text',
            text: 2,
          }]
        }, {
          type: 'view',
          children: [{
            type: 'plain-text',
            text: 3,
          }]
        }]
      }]
    }]
  }
}
```

# 缺点

- 微信平台如果要启用这个能力，必须保证没有递归使用组件。
- 启用特性后，Remax 生成的模板大小不再是个固定值，会随着项目增大而增大。

# 其他可选方案

无

# 其他问题

无
