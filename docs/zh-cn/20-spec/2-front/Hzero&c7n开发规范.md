# 目的
出于加快开发流程，提高代码质量，减少不必要的沟通和方便维护代码等目的，制定用于Hzero & Choerodon-UI 集成前端的开发规范。
以下主要关注于集成项目的一些规范开发说明，与Hzero 规范冲突时遵循此规范，Hzero 项目前端开发准备工作参考：
[HZERO 前端开发规范](http://hzerodoc.saas.hand-china.com/zh/docs/development-specification/font-development-specification/](http://hzerodoc.saas.hand-china.com/zh/docs/development-specification/font-development-specification)

# 目录结构

## 项目
```
|-- opt                     # 根目录
    |-- .babelrc
    |-- .editorconfig
    |-- .eslintignore
    |-- .eslintrc.js
    |-- .gitignore
    |-- .prettierignore
    |-- .prettierrc
    |-- .stylelintrc
    |-- README.md
    |-- jsconfig.json
    |-- lerna.json
    |-- package.json
    |-- yarn.lock
    |-- config                     # 配置文件夹
    |   |-- alias.js
    |   |-- c7nUIConfig.js
    |   |-- compileBuildEnv.js
    |   |-- compileStartEnv.js
    |   |-- compileTestEnv.js
    |   |-- env.js
    |   |-- paths.js
    |   |-- routers.js                     # 路由
    |   |-- theme.js                     # ui 变量覆盖文件
    |   |-- webpack.config.js
    |   |-- webpack.dll.config.js
    |   |-- webpackDevServer.config.js
    |   |-- jest
    |-- mock
    |-- public
    |-- scripts
    |-- src
        |-- index.js
        |-- index.less
        |-- router.js
        |-- serviceWorker.js
        |-- setupProxy.js
        |-- models                     # model 文件夹
        |-- routes                     # 界面文件夹
        |-- services                     # service 文件夹
        |-- utils                     # 公共方法 / 组件 文件夹
 ```
## 界面
```
|-- routes
|-- modules1
|-- detail                   # page1
|    |-- DetailPage.js          # 页面以 Page 为后缀
|    |-- InputFieldTable.js     # Table 以 Table 为后缀
|    |-- LogVIewModal.js        # 弹窗 以 Modal 为后缀
|--  list                     # page2
|    |-- ListPage.js            # 页面以 Page 为后缀
|-- stores                   # 存放 DataSetProps 配置文件
|-- xxxHeaderDS.js         # DataSetProps 以 DS 为后缀
|--  xxxLineDS.js
 ```

# 代码规范
## 基本风格
* 对复杂页面上，根据逻辑或位置块进行组件划分，不仅方便后期改造，定位bug，还能优化性能，不要把过多的代码全部写在一个文件里，或者写在一个方法里
* 页面组件，工具函数都放在本页面目录下，除了一些跨页面使用组件或公用组件，尽量达到“一个目录一个页面，可迁移可删除”的目的
* 使用async/await处理异步处理，如果要处理一些可能会出现的错误，使用try-catch进行包裹
* 注意多个异步情况下Promise.all的使用来避免请求阻塞，比如页面加载时要同时发多个请求，如果使用多个await，会导致后面的请求等待前面的请求完成才执行
* 使用classnames库来处理条件判断生成classname的情况，如果比较简单使用三元表达式

```
import classNames from 'classnames';

// simple
<li className={active ? 'active' : null} />

// complex
const classString = classNames(`${prefixCls}-form-editor`, {
  dragging,
});
<li className={classString} />
```
* 根据项目，使用query-string库来处理url请求中的数据获取情况
* 提供的lint（Eslint 、 styleLint）处理代码
* 必须配置husky进行检查，以在commit前触发代码检查，不通过的代码将无法提交
* URL参数命名注意不要与层级参数organizationId，id，type，name等同名
## 命名规范
* 文件夹命名一律小写，使用-来连接（不用驼峰）
* DataSetProps 文件以 DS 为后缀
* 页面以 Page 为后缀
* 组件以对应组件名为后缀
* 函数命名
>能够表明这个函数的用处。

```
// bad 
handleSave() {}

// good，可以带上模块名称
handleSaveCompany() {}
```
  * 处理事件，handle起头
    * 如处理用户输入, handleChangeInput()
    * 处理点击按钮事件, handleClickBtn()
    * handleFilter()
    * handleSelect()
  * get方法，用于获取部分参数,渲染体或者dom节点
    * getMaxNumber()
    * getDefaultSelection()
    * getRecordKey()
    * getOption()
    * getData()
    * getFooterContent()
    * getPopuoContainer()
  * render方法，把较为独立的部分拆分出来，方便定位和修改逻辑
    * renderList()
    * renderHeader()
    * renderList()
  * 自定义事件名要求带有强烈的语义性,如
    * isSorted()
    * isFilterChanged()
    * hasPagination()
    * resetData()
    * focus()
    * blur()
    * saveData()
* 变量命名
>能够表明这个变量的用处

```
// bad
visible: false

// good
modalVisible: false
```
* 组件属性命名

```
// bad
  <TableList
    search={this.handleSearchCalendar}
  />

// good
  <TableList
    onSearch={this.handleSearchCalendar}
  />
```
## 注释规范
* 头文件注释

```
/**
 * module - 
 * @Author: xxx <xxx@hand-china.com>
 * @Date: 2019-08-14 14:31:25
 * @LastEditTime: 2019-08-14 17:34:26
 * @Copyright: Copyright (c) 2018, Hand
 */
```

* 其他注释规范参考[Hzero](http://hzerodoc.saas.hand-china.com/zh/docs/development-specification/font-development-specification/specification/comment/）

## git提交规范

```shell
[操作][:][空格][commit内容]
```

`[commit内容]`请详细填写具体的文件新增/修改/删除操作过程

例如

```shell
fix: 修复查询功能bug
```

* 操作标识符

```shell
fix：修复bug
feat：更新/新增文件/新特性
modify：重命名
delete：删除文件
docs: 文档调整补充
```

patches:

```bash
$ git commit -a -m "fix(parsing): fixed a bug in our parser"
```

features:

```bash
$ git commit -a -m "feat(parser): we now have a parser \o/"
```

breaking changes:

```bash
$ git commit -a -m "feat(new-parser): introduces a new parsing library
BREAKING CHANGE: new library does not support foo-construct"
```
other changes:

You decide, e.g., docs, chore, etc.

```bash
$ git commit -a -m "docs: fixed up the docs a bit"
```

## CSS/LESS相关
* 样式文件统一使用less， 原来使用sass（scss）、css的一律改为less
* 所有颜色值使用变量，尤其是主题色或主题色相关的，必须使用@primary-color方便后期进行主题替换
* 所有px单位改为rem，计算方式为px/100
* css禁止使用html元素选择器，允许子选择器使用html选择器
* 覆盖ui库的样式时，需要引入@c7n-prefix或@c7n-pro-prefix变量
## Table
* 可编辑表格新建触发添加行加在第一行
* 表格行内操作按钮使用 commands 属性，其余情况可使用 rerender 处理
## DataSet
* events 事件处理
  * 事件涉及业务逻辑较多，代码量大时单独文件处理
  * 代码量少，事件置于 DataSet 文件内或界面实例处均可
  * 字段必输、校验等可使用 dynamicProps 属性， 若非实时处理以及简单的字段更新变动等，可置于 update 事件回调中处理
* 原先的零散状态管理，如分页排序、loading与否等，可以用一个dataset来进行管理
* dataset在组件内部实例化，stores文件夹中的文件是dataset的配置文件，暴露一个plain object或者返回值为plain object的函数，参数接收部分通过调用时传进去的值，比如intlPrefix
* 如果只是简单的增删查改操作，使用transport完成api的管理，下面代码只是个例子，如果返回的结果不是带rows（猪齿鱼默认是list）的对象，需要将dataKey设为其他对应数据集的字段，如果返回的结果本身就是数组或者只是代表数据集中第一条数据的对象时，需要将dataKey设为null，更多请查看猪齿鱼官网，ds相关以及相关全局配置。

```
{
    dataKey: null,
    transport: {
      read: {
        url: `/lc/v1/organizations/${orgId}/view/${code}`,
        method: 'get',
      },
    },
}
```
## Form
* 涉及头行结构表单保存时，isModify 方法 noCascade 确认参数，默认是会校验行
* const result = await ds.submit();  如果result为undedined，就是没有数据变更

