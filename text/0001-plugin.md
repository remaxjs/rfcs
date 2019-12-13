- 日期: 2019-12-12
- RFC PR:
- Issue:

# 总结

引入插件机制，通过插件机制可以增强 Remax 的功能，包括但不限于:

- 支持多平台
- 扩展语法支持能力
- 切换渲染引擎，如 rax
- 自定义打包行为 （切换 webpack？）
- 自定义 CLI 命令
- 扩展平台功能
- ...

# 基本例子

待补充

# 背景

Remax 目前的代码与平台有一定的耦合，我们要做到 remax 和 remax-cli 与平台无关。
Remax 目前支持跨平台的能力不强。
并且考虑到用户需求的灵活性，各平台的差异性，要有足够自由的空间给插件发挥。

Remax 的编译时与运行时流程图如下：

```
╔═══════════════════════════╗
            编译时
╚═══════════════════════════╝
┏━━━━━━━━━━━━━━━┓
1. 处理 CLI 命令
┗━━━━━━━━━━━━━━━┛
┏━━━━━━━━━━━━━━━━━━┓
2. 处理 RemaxConfig
┗━━━━━━━━━━━━━━━━━━┛
┏━━━━━━━━━━━━━━━━━━┓
3. 引入 adapter 代码
┗━━━━━━━━━━━━━━━━━━┛
┏━━━━━━━━━━━━━━━━━━━━┓
4. 处理 Rollup Config
┗━━━━━━━━━━━━━━━━━━━━┛
  ┣━━━━━━━━━━━━━━━━━━━┓
  4.0 获取 entry 文件
  ┗━━━━━━━━━━━━━━━━━━━┛
  ┏━━━━━━━━━━━━━━━━━━━┓
  4.1 处理 Rollup 插件
  ┗━━━━━━━━━━━━━━━━━━━┛
    ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.1 清理 output 目录
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.2 copy native 目录
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.3 alias
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.4 url
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.5 node resolve
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.6 commonjs
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.7 stub
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.8 babel [收集 native 组件，处理 page 页面]
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
      ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
      4.1.10.1 引用 babel-preset-remax
      ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.9 babel [收集 native 组件，处理 app 页面]
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.10 babel [收集 native 组件，收集需要生成的原生模板]
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
      ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
      4.1.10.2 处理 createHostComponent 收集到的 host 组件
      ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.11 postcss
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
      ┣━━━━━━━━━━━━━━━━━┓
      4.1.11.1 px 转 rpx
      ┗━━━━━━━━━━━━━━━━━┛
      ┏━━━━━━━━━━━━━━━━━┓
      4.1.11.2 url
      ┗━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.12 处理 json 文件
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.13 处理环境变量
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.14 文件重命名
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.15 去除路径中的 src
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.16. fix regenerateRuntime
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.17 收集 native 组件引用
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.18 生成原生文件
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    4.1.19 显示 build 进度
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
  ┏━━━━━━━━━━━━━━━━━━━━━━━┓
  4.2. 处理 Rollup Options
  ┗━━━━━━━━━━━━━━━━━━━━━━━┛
┏━━━━━━━━━━━━━━━━━━━┓
5. 处理 watch
┗━━━━━━━━━━━━━━━━━━━┛
┏━━━━━━━━━━━━━━━━━━━┓
6. rollup build
┗━━━━━━━━━━━━━━━━━━━┛
┏━━━━━━━━━━━━━━━━━━━┓
5. rollup write
┗━━━━━━━━━━━━━━━━━━━┛


╔═══════════════════════════╗
            运行时
╚═══════════════════════════╝
┏━━━━━━━━━━━━━━━┓
1. 创建 App 对象
┗━━━━━━━━━━━━━━━┛
  ┣━━━━━━━━━━━━━━━━━━━━━┓
  1.1 注册App生命周期
  ┗━━━━━━━━━━━━━━━━━━━━━┛
  ┏━━━━━━━━━━━━━━━━━━━━━┓
  1.2. 创建 AppContainer
  ┗━━━━━━━━━━━━━━━━━━━━━┛
  ┏━━━━━━━━━━━━━━━━━━━━━┓
  1.3. 创建 React 实例
  ┗━━━━━━━━━━━━━━━━━━━━━┛
    ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    1.3.1 注册 HostConfig
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    1.3.2 创建 React root Container
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
┏━━━━━━━━━━━━━━━┓
2. 创建 Page 对象
┗━━━━━━━━━━━━━━━┛
  ┣━━━━━━━━━━━━━━━━━━━━━┓
  2.1 注册Page生命周期
  ┗━━━━━━━━━━━━━━━━━━━━━┛
  ┏━━━━━━━━━━━━━━━━━━━━━┓
  2.2. Page render
  ┗━━━━━━━━━━━━━━━━━━━━━┛
    ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    2.2.1 创建 Container 实例
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    2.2.2 创建 React Element 实例
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    2.2.3 创建 Portal
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    2.2.4 更新 React root Container
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
┏━━━━━━━━━━━━━━━━━┓
3. React renderer
┗━━━━━━━━━━━━━━━━━┛
  ┣━━━━━━━━━━━━━━━┓
  3.1 hostConfig
  ┗━━━━━━━━━━━━━━━┛
    ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    3.1.1 hooks 触发 Container 行为
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
      ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
      3.1.1.1 更新虚拟 dom
      ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
      ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
      3.1.1.2 requestUpdate, update 入列
      ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
      ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
      3.1.1.3 applyUpdate
      ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
        ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
        3.1.1.3.1 props 别名
        ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
        ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
        3.1.1.3.2 创建 update action
        ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
        ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
        3.1.1.3.3 执行原生 setData
        ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
      ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
      3.1.1.4 clearUpdate，停止更新行为
      ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    3.1.2 处理 props
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
      ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
      3.1.2.1 处理 SyntheticEvent 逻辑
      ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```

# 详细设计

## 创建 Remax 插件

开发者

```ts
// plugin/node.js
export default function alipayNodePlgin() {
  return {
    ...
  }: RemaxNodePluginConfig
});

// plugin/runtime.js
export default function alipayRuntimePlgin() {
  return {
    ...
  }: RemaxNodePluginConfig
});
```

## 使用 Remax 插件

用户

```ts
// remax.config.ts
export default {
  plugins: [
    'remax-plugin-alipay', // 编译时自动查找 npm 包 remax-plugin-alipay/node.js, 运行时自动插找 remax-plugin-alipay/runtime.js
    'aliay', // 自动映射成 remax-plugin-alipay
    ...
  ],
}
```

Remax

```ts
// remax-cli
export { usePlugin };

// 读取 remax.config.js，将插件引入
plugins.forEach(plug => {
  usePlugin(plug);
});
```

## PluginContext

```ts
// 首先我们要有一个 PluginContext 供创建 Plugin 的全过程可以访问到需要的东西
// 所有 Plugin hook 都可以通过 this.context 拿到 PluginContext

// 运行时相关
interface RemaxRuntimePluginContext {
  ...
}

// 编译时相关
interface RemaxNodePluginContext {
  ...
}
```

## PluginConfig

```ts
// 定义实现 plugin 要实现的 options, hooks

// 运行时相关
interface RemaxRuntimePluginConfig {
    ...
}

// 编译时相关
interface RemaxRuntimePluginConfig {
    ...
}
```

## 扩展 CLI 命令 - 编译时 1.

开发者

```ts
// plugin/node.ts
export default function plugin() {
  return {
    /**
     * 将 yargs 对象传入，返回扩展后的 yargs 对象
     * @param  yargs
     */
    extendsCLI: cli => cli
  };
}

// 同时 this.context.cliOptions 可以访问到用户输入的 CLI 参数
```

## 扩展 RemaxOptions - 编译时 2.

开发者

```ts
// plugin/node.ts
export default function plugin() {
  return {
    /**
     * 扩展 remaxOptionsSchema 定义
     */
    extendsRemaxOptionsSchema: remaxOptionsSchema => remaxOptionsSchema,

    /**
     * 自定义 remaxOptions
     * @param defaultRemaxOptions: remax options 默认值
     * @param defaultRemaxOptions: remax options 默认值
     */
    mapRemaxOptions: (defaultRemaxOptions, inputRemaxOptions) => remaxOptions
  };
}

// 同时 this.context.remaxOptions 可以访问到最终的 remax options 参数
```

_结合 CLI 和 RemaxOptions 的扩展能力，开发者可以自动把用户选择的平台对应的 plugin 引入到 remax.config.js 中，这样用户可以无感知地使用 remax-plugin-\${plaform}，无需手动引入_

## 废弃 <引入 Adapter 代码 编译时 3.>

## 扩展 RollupConfig 编译时 4.

开发者

```ts
// plugin/node.ts
export default function plugin() {
  return {
    /**
     * 完全接管 rollupConfig 做调整
     * remax 现有的 rollupConfig
     * @param  rollupConfig
     */
    extendsRollupConfig: rollupConfig => rollupConfig
  };
}

// 同时 this.context.rollupConfig 可以访问到最终的 rollupConfig
```

_通过扩展 RollupConfig，开发者可以自定义 rollup 行为_

## 自定义 Entry 文件. 编译时 4.0

开发者

```ts
// plugin/node.ts
export default function plugin() {
  return {
    /**
     * 自定义 entry 文件
     */
    getEntreis: () => any
  };
}
```

## 扩展 Rollup Plugin. 编译时 4.1

开发者

```ts
// plugin/node.ts
export default function plugin() {
  return {
    /**
      * 帮助开发者扩展已有 rollup 插件的参数
      */
    extendsRollupPlugins: () => ([
      {
        name: 'plugin-name',
        options: {
          ...
        } || (originalOptions) => ({ ... })
      }
    ]),
  }
};

// 同时 this.context.rollupConfig 可以访问到最终的 rollupConfig
```

Remax

```ts
// rollupConfig.ts
plugins.map(plug => {
  const customOptions = remaxPlugin.extendsRollupPlugins.findByName(plug.name)
    .options;
  return plug(customOptions || originalOptions);
});
```

_通过扩展 RollupConfig，开发者可以自定义 rollup 行为_

## 废弃 <babel-plugin-stub 编译时 4.1.7>

## 自定义处理 page 页面行为 编译时 4.1.8

开发者

```ts
// plugin/node.ts
export default function plugin() {
  return {
    /**
     * 自定义处理 page 组件的 babel 插件
     */
    customBabelPluginPage: () => plugin
  };
}
```

## 自定义处理 app 页面行为 编译时 4.1.9

开发者

```ts
// plugin/node.ts
export default function plugin() {
  return {
    /**
     * 自定义处理 app 组件的 babel 插件
     */
    customBabelPluginApp: () => plugin
  };
}
```

## 自定义收集 native 组件行为 编译时 [4.1.8, 4.1.9, 4.1.10]

开发者

```ts
// plugin/node.ts
export default function plugin() {
  return {
    /**
     * 自定义处理 原生组件的 babel 插件
     */
    customBabelPluginNativeComponent: () => plugin
  };
}
```

## 自定义收集 JSX 标签的行为 编译时 4.1.10

开发者

```ts
// plugin/node.ts
export default function plugin() {
  return {
    /**
     * 自定义收集 jsx 标签的 babel 插件
     */
    customBabelPluginJSXTag: () => plugin
  };
}
```

## 配置原生文件扩展名 编译时 4.1.14

开发者

```ts
// plugin/node.ts
export default function plugin() {
  return {
    extensions: {
      template: 'axml',
      style: 'axss',
      jsHelper: 'sjs'
    }
  };
}
```

## 注册 App 生命周期 运行时 1.1

开发者

```ts
// plugin/runtime.ts
export default function plugin() {
  return {
    /**
      * 注册生命周期
      */
    registerAppLifeCycle: (options) => any;
  }
}
```

## 注册 App 生命周期 运行时 1.1.

开发者

```ts
// plugin/runtime.ts
export default function plugin() {
  return {
    /**
      * 注册 App 生命周期
      */
    registerAppLifeCycle: (options) => any;
  }
}
```

## 注册 Page 生命周期 运行时 2.

开发者

```ts
// plugin/runtime.ts
export default function plugin() {
  return {
    /**
      * 注册 Page 生命周期
      */
    registerPageLifeCycle: (options) => any;
  }
}
```

## 扩展 VNode 运行时 3.

开发者

```ts
// plugin/runtime.ts
export default function plugin() {
  return {
    /**
      * 扩展虚拟 dom 类
      */
    extendsVNode: (VNode) => VNode;
  }
}
```

## 处理 props 运行时 3.1.2

开发者

```ts
// plugin/runtime.ts
export default function plugin() {
  return {
    /**
      * hostConfig 处理 props before hook
      */
    beforeHostConfigProcessingProps: (...propsOptions) => props,
    /**
      * hostConfig 处理 props after hook
      */
    afterHostConfigProcessingProps: (...propsOptions) => props;
  }
}
```

remax

```ts
// hostConfig.ts
...

processProps(newProps, ...options) {
  const props = beforeHostConfigProcessingProps(newProps, ...options);
  ...
  return afterHostConfigProcessingProps(props, ...options);
}

...

```

## 扩展 Synthetic Event 3.1.2.1

开发者

```ts
// plugin/runtime.ts
export default function plugin() {
  return {
    /**
      * 扩展 synthetic 事件池
      */
    extendsSyntheticPool: (pool) => pool,
    /**
      * 扩展 synthetic 事件
      */
    extendsSyntheticEvent: (event) => event;
  }
}
```

## 扩展 Container 3.1.1.3.2

开发者

```ts
// plugin/runtime.ts
export default function plugin() {
  return {
    /**
      * 创建 update action
      */
    createContainerUpdateAction: (container) => action,
    /**
      * 已停止更新 hook
      */
    containerDidStopUpdate: (container) => void;
  }
}
```

# 缺点

代码冗余会有一定程度上升，维护多平台的成本上升。

# 其他可选方案

无

# 其他问题
