- 日期: 2019-12-10
- RFC PR: https://github.com/remaxjs/remax/pull/470 
- Issue: 

# 总结

我们将 remax 和 remax-cli 进行重构，做到与平台无关。所有与平台相关的代码都用 adapter 接口 的形式执行。
比如 remax-cli 中，每个平台会有自己的 template 模板，可以改成 adapter.useTemplate()，而具体使用的是哪个模板，由平台自己配置 adapter 实现逻辑。

# 基本例子

adapter 添加 adapterConfig.js 文件

```ts
// adapter/alipay/adapterConfig.js
import { useAdapter } from 'remax';

useAdapter({
  platform: 'alipay', // 指定平台名称
  containerPrepareUpdateAction() { // 准备更新所需的 action
    ...
  },
 // 其他需要实现的接口
  ...
})

```

开发者依然从 `remax/alipay` 导入使用.

```ts
import { View } from 'remax/alipay';
```

# 背景

为了区分各个平台的代码和实现，我们需要做到 remax 和 remax-cli与平台解耦。比如 alipay 平台我们希望能把 React Component 转换成原生自定义组件，这一特性可能就是针对 Alipay 来做的。

# 详细设计

## Remax 流程图与 AdapterConfig 执行关系

### 编译阶段

1. 读取 RemaxConfig
    * `extendsRemaxConfigOptions`
2. 构建 Rollup Config
    * `extendsRollupConifg`
3. 执行 Rollup 编译
    1. 获取 entries
        * getEntries
    2. 清理output目录
    3. node resolve 处理
    4. commonjs 处理
    5. babel 处理
        1. 收集需要创建的模板参数（一般是组件）
            * `createPluginForCollectTemplatesParams`
        2. 收集需要创建的原生模板参数
            * `createPluginForCollectNativeTemplatesParams`
    6. postcss 处理
    7. 环境变量处理
    8. 重命名文件
        * `extensions.style`
    9. 原生组件处理
        * `createPluinForCollectNativeTemplatesParams`
    10. 生成原生文件
        * `createPluginForGenerateTemplateFiles`

### 运行时

1. 创建 App 对象
    * 注册生命周期
        * `appRegisterLifeCycle`
2. 创建 Page 对象
    * 注册生命周期
        * `pageRegisterLifeCycle`
3. 进入 component 逻辑
4. 进入 hostConfig 逻辑
    1. 处理 SyntheticEvent 逻辑
        1. 创建事件代理
        2. 用户触发点击事件
        3. 是否是新事件
            * `isNewNativeEvent`
        4. 创建新事件流或按冒泡事件处理
            * `getIdentityFromNativeEvent`
  2. 进入 Container requestUpdate 逻辑
      1. update 操作进入队列
      2. update 队列通过 setData 进行更新
          * `containerPrepareUpdate`
      3. 停止 update
          * `containerDidStopUpdate`
5. 其他
    * createHostComponent
    * createLifeCycleHook
    * createPromisifyAPI

## AdapterConfig 接口定义

```ts
interface AdapterConfig {
  /**
   * 平台名称
   */
  platform: string;
  // ************** Container 相关 **************
  /**
   * Container 执行虚拟 dom update 的生命周期，返回 update 所需的 action
   * @param container Container
   */
  containerPrepareUpdate: (
    container: Container
  ) => {
    action: any;
  };
  /**
   * Container 停止 update 行为的生命周期
   * 返回值为 boolean 类型，决定是否停止
   * @param container Container
   */
  containerDidStopUpdate: (container: Container) => void;
  // ************** createAppConfig 相关 **************
  /**
   * 注册 app 上的原生生命周期
   * 返回一个 object，键值对为 生命周期名称 => 注册的方法，如：
   * {
   *   onShow(options) {
   *     registerLifeCycle('onShow', options)
   *   }
   * }
   */
  appRegisterLifeCycle: (
    registerLifeCycle: Function
  ) => {
    [key: string]: Function;
  };
  // ************** createPageConfig 相关 **************
  /**
   * 注册 page 上的原生生命周期
   * 返回一个 object，键值对为 生命周期名称 => 注册的方法，如：
   * {
   *   onShow(options) {
   *     registerLifeCycle('onShow', options)
   *   }
   * }
   */
  pageRegisterLifeCycle: (
    registerLifeCycle: Function
  ) => {
    [key: string]: Function;
  };
  // ************** SyntheticEvent 相关 **************
  /**
   * 判断是不是新发起的原生事件
   */
  isNewNativeEvent: (event: any) => boolean;
  /**
   * 根据原生事件生成唯一 SyntheticEvent ID
   */
  getIdentityFromNativeEvent: (event: any) => string;
  // ************** remax.config 相关 **************
  /**
   * 扩展 remax.config 选项
   * @param options 初始选项
   * @return 返回扩展后的选项
   */
  extendsRemaxConfigOptions: (options: {
    [key: string]: any;
  }) => { [key: string]: any };
  // ************** rollupConfig 相关 **************
  /**
   * 扩展 rollup config
   */
  extendsRollupConfig: (rollupConfig: any) => any;
  // ************** getEntries 相关 **************
  /**
   * 读取所有相关入口文件
   *
   * @memberof AdapterConfig
   */
  getEntries: () => any;
  // ************** babel & rollup 插件相关 **************
  /**
   * 创建 babel 插件，用于编译时收集需要生成的模板参数
   */
  createPluginForCollectTemplatesParams: () => Function;
  /**
   * 创建 rollup 和 babel 插件，用于编译时收集需要生成的原生组件模板参数
   */
  createPluginForCollectNativeTemplatesParams: () => [Function, Function];
  /**
   * 创建 rollup 插件，用于创建原生文件
   */
  createPluginForGenerateTemplateFiles: () => [Function, Function];
  /**
   * 配置原生文件的后缀名
   */
  extensions: {
    style: string;
  };
}
```

## 帮助方法
在 AdapterConfig 之外，还有一些帮助方法：

```ts
// 帮助平台创建生命周期 hook
createLifeCycleHook: (callback: Function) => Fucntion
// 帮助创建 host 组件
createHostComponent: (name: string, props: {}) => React.ComponentClass
// 帮助创建 promise 化的 api
createPromisifyAPI: (api: any) => any;
```

除了注册 AdapterConfig，平台还应该导出
  * api
  * react host 组件
  * react hook
  * type 定义

## 注册 adapter 的时机

在 remax-cli 的 rollup 编译中，注入代码 import `'remax/${platform}/adapterConfig'` 到 entry 文件。


# 缺点

代码冗余会有一定程度上升，同样的功能在多个平台需要重复测试。

# 其他可选方案

无

# 其他问题

要尽量考虑各个平台的差异，比如微信要收集 jsx 片段，支付宝要将 React 组件转换成自定义组件，提供高灵活性的接口。
