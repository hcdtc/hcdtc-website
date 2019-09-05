# Hzero&c7n初版规范整理

## 部分代码规范

- 可编辑表格新建触发添加行加在第一行
- 表格行内操作按钮使用 commands 属性，其余情况可使用 rerender 处理
- DataSet events 事件处理
  - 事件涉及业务逻辑较多，代码量大时单独文件处理
  - 代码量少，事件置于 DataSet 文件内或界面实例处均可
  - 字段必输、校验等可使用 dynamicProps 属性， 若非实时处理以及简单的字段更新变动等，可置于 update 事件回调中处理
- 涉及头行结构表单保存时，目前需要使用 isModify 方法校验头行 DataSet, 确认表单是否改变进而触发提交事件
- 对复杂页面上，根据逻辑或位置块进行组件划分，不仅方便后期改造，定位bug，还能优化性能，不要把过多的代码全部写在一个文件里，或者写在一个方法里
- 渲染类函数使用render开头，比如renderTable，renderItems，renderHeader
- 事件处理类函数（由页面直接调用的函数）使用handle开头，比如handleClick
- 工具函数使用get，set等开头
- 使用async/await处理异步处理，如果要处理一些可能会出现的错误，使用try-catch进行包裹
- 注意多个异步情况下Promise.all的使用来避免请求阻塞，比如页面加载时要同时发多个请求，如果使用多个await，会导致后面的请求等待前面的请求完成才执行
- 使用classnames库来处理条件判断生成classname的情况，如果比较简单使用三元表达式
- 使用query-string库来处理url请求中的数据获取情况
- 引用其他文件时，不写以jsx等结尾的后缀，因为编译后jsx文件不存在（被编译为js）会导致找不到文件而报错
- URL参数命名注意不要与层级参数organizationId，id，type，name等同名
- 所有颜色值使用变量，尤其是主题色或主题色相关的，必须使用@primary-color方便后期进行主题替换
- 所有px单位改为rem，计算方式为px/100
- css禁止使用html元素选择器，允许子选择器使用html选择器
- 覆盖ui库的样式时，需要引入@c7n-prefix或@c7n-pro-prefix变量：

```
说明
处理事件，handle起头

如处理用户输入, handleChangeInput()
处理点击按钮事件, handleClickBtn()
handleFilter()
handleSelect()
get方法，用于获取部分参数,渲染体或者dom节点

getMaxNumber()
getDefaultSelection()
getRecordKey()
getOption()
getData()
getFooterContent()
getPopuoContainer()
render方法，把较为独立的部分拆分出来，方便定位和修改逻辑

renderList()
renderHeader()
renderList()
自定义事件名要求带有强烈的语义性,如

isSorted()
isFilterChanged()
hasPagination()
resetData()
focus()
blur()
saveData()
对于函数名看不出具体含义，或者逻辑比较复杂的，写上注释，并且列举可能情况方便修改
```

## 目录结构

```
| moudleName
|  └─list
│  | └─index.js
│  | └─index.css
|  | └─List.js
|  └─detail
|  | └─Detail.js
|  | └─index.js
|  └─stores
|  | └─XxxDS.js
|  | └─XxDS.js
|  | └─index.js
|  └─style
│    └─index.css
```

- 无全局样式, style 放置对应界面文件夹下
- 文件夹命名一律小写，使用-来连接（不用驼峰），常用的包括（routes：表示按菜单或路由划分的模块，locale：多语言处理，components：跨页面使用的组件，utils：公用工具函数等），组件命名大写开头，使用驼峰，工具类命名小写开头，使用驼峰
- DataSet 使用帕斯卡命名

## 待确认规范

- 页面后缀
- index 导出文件
