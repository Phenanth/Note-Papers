# Record

## 每周绩效报告模板

项目名称：绩效管理系统；

任务名称：绩效管理系统项目开发； 

 上周任务：绩效管理系统项目维护及2.0规划；

本周任务：绩效管理系统项目新版本需求评审及项目开发。

## 0902

### 入职安排

- ~~户籍证明~~
- ~~实习文函~~
- ~~开户~~
- ~~问一下工作安排~~

### 工作安排

- ~~了解源代码结构~~
- ~~根据`router.js`分析页面关系~~
- 分析数据流
- 看需求文档
### 学习安排

- less
- element-ui
- /src/main.js语句意思，主要是和Vue有关的

## 0905

1. dropdown还是用不了
2. 试用了el-link, el-button["type"="text"]，el-card之类的
3. 之后想要试一下selector里面的【创建条目】那种类型的选择框以及【Tag标签】里面的可移除标签来做一下自己那个系统里面的标签【个人】
4. Loading加载可以考虑用在加载时候【个人】
5. 下午尝试实现配合后端的登录页面，完成了于登录页面相关的axios和vuex：store在main.js的引用方法、api单独引用要用模块的写法

工作安排：

1. 看需求文档
2. 分析数据流
3. 详细的页面元素分析

学习安排：

1. less
2. element-ui

## 0906

1. 登出功能相关的js和vuex
2. 路由守护：Router添加总体的路由钩子的写法，修改NaviagationDuplicated报错：https://github.com/vuejs/vue-router/issues/2881#issuecomment-520554378
3. vuex：Cookies的Expires属性相关内容
4. 继续熟悉API调用，添加Account总体数据显示

## 0909

1. seletor的远程加载：防抖和节流
https://juejin.im/post/5c6bab91f265da2dd94c9f9e
https://blog.csdn.net/sinat_17775997/article/details/73942828 vue全局函数
2. 正则
http://www.regexlab.com/zh/regref.htm
https://blog.csdn.net/lxcnn/article/details/4304754 环视（预检查）
https://regex101.com/
3. 关于返回顶端按钮的使用方法，手册写得不清楚：https://github.com/ElemeFE/element/issues/16051
4. less http://lesscss.cn/features/#features-overview-feature

## 0912

用到：

- validator，要用`$refs`对要检查的form注册后才能在方法里面检验它的数据
- 用vuex在组件之间转存查询得到的数据（其实也可以考虑用一个Vue当bus）
- element，相关组件的使用与熟悉
- axios，里面`${}/order/issue/`路径一定要写最后一个`/`
- 对Date对象原型添加全局可使用的函数，用`export default install(Vue, options)`实现了。

未完成：

- 没有税号就没办法查流水，但是要有税号才能查公司相关的信息
- 不知道是因为框架还是什么原因，点击单个item没办法直接切换整个元素的选中
- 单选框样式（radio）（缺背景图）
- 页面整体的字体：复制（复制了暂时也没用）

## 0916

基本做了一些小程序的界面和功能点上的优化。

CSS：

头像图片样式更改（`:src`内容从网页链接变成`require('../assets/logo.png')`）
单选框样式更改（改为背景图片，记得要用`background-size: cover/100%`，切换使用`:class`实现）
消息框的样式调整（输入框`clearable`，消息框直接更改`el-message`的CSS样式）

js：

把时间格式的函数放到全局函数文件里（`exports.install = ...`）
开票完了之后跳转新的页面（`window.location.href = url`）
查询页面的公司信息通过URL获取税号并查询取得（`let e = /(?=taxCode=)(\d+)/, let taxCode = e.exec(url)[1]`）

未完成：

暂时不知道访问程序的完整URL是什么，所以无法得知能够获取`taxCode`的地址如何获得

## 0917

（后来补的）

接口
http://192.168.100.11:8093/cancer-invoice/swagger-ui.html#/order-controller/queryOrderUsingPOST

CSS：
头像图片更改
单选框样式更改
消息框的样式调整
莫名其妙的颜色问题
打包后静态图片失效，可能会有用：https://blog.csdn.net/jian_xi/article/details/70158952
nginx的配置文件：https://blog.csdn.net/zhongzh86/article/details/70173174
修改了一些element的默认css之后webpack一打包又没了，用!important覆盖掉了。

js：

配置：webpack.base.conf.js的resolve里面添加'static'

## 0925

> 学了一下python下面的django服务器，[参考网站](https://docs.djangoproject.com/en/2.2/)

```bash
mysqld -install (mysqld -remove)
net start mysql
```

首先修改settings.py，添加应用路径
更改安装应用之后，进行模型迁移

```bash
python manage.py makemigrations polls 
python manage.py migrate
```

关于`Model.objects.get()`和`filter()`的区别：返回的是`question`还是`questionSet`的区别

## 1008

了解关于vue3.0的更新内容
https://juejin.im/post/5d996e3e6fb9a04e3043cc5b，Vue 3 原理剖析：数据响应系统
https://zhuanlan.zhihu.com/p/68477600，Vue Function-based API RFC：将 2.x 中与组件逻辑相关的选项以 API 函数的形式重新设计。

## 1009

感冒休息一天

## 1010
继续看了Vue3的源码部分，reactive以及effect的ts代码看了一下。（做了部分笔记，画了图）
之后又看了一下关于Element-UI里面样式的部分：https://juejin.im/post/5d9ebaddf265da5b591b64b3，还没看完

https://www.tslang.cn/docs/handbook/typescript-in-5-minutes.html，Typescript 五分钟快速教程

国庆过来的周末（周五，11号？）会有公司组织的电影
## 1011
签自己的名字就先打卡打掉
签组长的名字就晚上提交考勤异常申请
https://vue-composition-api-rfc.netlify.com/#detailed-design：关于官方写的对组合（Composition）API的介绍

## 1014
了解去除了v-model之后，使用reactive进行数据响应的原理：https://juejin.im/post/5d9c6b90e51d4578331cbd0c，讲到了proxy过的对象背后存在的set和get两种对赋值操作行为响应的方法，这里知识点对应的源代码（中文注释有部分错误）：https://github.com/KieSun/vue-interpretation/blob/comment/packages/reactivity/src/reactive.ts

## 1015
https://juejin.im/post/5c08f3756fb9a049b7802a2d，proxy对执行速度和内存占用优化的原理（利用es6的新数据结构，以及Reflect.get和Reflect.set可以自动监听数据对应变化，而非对每个属性进行遍历获取和遍历跟踪来减少开销）以及根据更新后的proxy实现observable

## 1016
继续看了一些文章
https://juejin.im/post/5da40462f265da5baf410a11，关于redis里面的geo组件如何实现查找附近的人的原理，其中重点介绍添加对象和寻找指定坐标附近对象。
Add：

1. 在存储中会检验并提取参数（经纬度）
2. 将经纬度替换成base52的哈希编码（geohash，有序的）
3. 存储到zset有序集合（跳表）里
Radius：
1. 首先将目标对象作为所在区域中心点，利用半径（附带地图精度）计算范围圆最多覆盖的附近八个方块区域（为了解决边界情况）
2. 计算八个哈希存储区域的所有地址对象与中心点距离
3. 比较筛选出满足在范围内的点（基于有序集合的跳表数据结构进行查询，这个查询的平均时间复杂度为O(logn)）
4. 作为附近人的member list返回

重要：ES6总结：https://juejin.im/post/5d9bf530518825427b27639d

## 1023
马尔科夫随机场很好的一篇文章，https://blog.csdn.net/polly_yang/article/details/9716591，讲了机器视觉方面的一些课题方向

## 1025
确认一下图片的显示形式

## 1104

> 姑且当周报了（周报是wya帮忙写的），这周从周一开始写广西的票据交付平台
>
> 心得：在有设计稿的时候还要去请求一份需求报告

1. 首页点击输入框不应该缩放页面：user-scalable=no
2. pdf查看器背景是黑色的
3. time-picker的样式在部署和服务器上不同
4. 将localhost换成指定IP，使用电脑共享wifi在手机上进行测试

1. 限制只能输入数字和大写字母？
2. 不要open了，还是image—》image刷新时间好长，需要一个加载页面？

注意：首页选择框的文字显示和输入框的文字显示不对齐


https://blog.csdn.net/yuan892173701/article/details/8731490，中文符号unicode（部分
https://baike.baidu.com/item/Unicode%E5%AD%97%E7%AC%A6%E5%88%97%E8%A1%A8/12022016，半角符号的unicode

号码 11 收款人 111
十条以上
2312312312，1111

1101: 13:42
决定修改：

1. [x] 按钮改成提供样式：Result页面两处与Index一处
2. [x] 去掉文字加粗样式：Result页面
3. [x] 去掉一个上方的框：Home页面
4. [x] 图片变糊的内容：用2x切图按比例缩放
5. [x]点击一次后按钮disable掉
6. [x] 色值的统一：按钮，字色全部统一
7. [x] 位数限制
8. [x] 提交票据表单的按钮下方加margin
9. [ ] email的格式检查

决定不修复：

1. 输入内容会被键盘挡到：个人设备的关系



## 1226

preview里面直接传过来的PNG图片贴近网页的方法？？[ok]直接把调用的url放在src里
info页面还有已收款和未收款的样式区别需要实现[ok]
确认一下verify的未收款筛选方法[ok]前端就这样，后端会给默认值
以及开票状态的对应数据情况
why无法发起核销[wait]等待服务器给核销状态配默认值

## 周报

### 0902周报

项目名称：绩效管理系统；

任务名称：绩效管理系统项目开发的进度跟进与学习； 

上周任务：绩效管理系统项目v1.0的原型、后台接口与源码的学习；

其中包括：

- `Less`、`Element-UI`的相关语法和组件的学习。
- 关于`Vuex`和`Axios`在项目中的运用的熟悉工作。
- 在以上学习内容的基础上对v1.0系统进行再现的模拟

本周任务：绩效管理系统项目的进度跟进与新系统开发的见习。

- 基于上一周的基础对相关技术展开进一步的学习，包括：
  - `正则`的使用
  - `Element-UI`的更多组件的用法
  - 尝试对更多功能点的实现的再现以及调用相应的后端服务等。

- 希望在不干扰组员对新系统开发的情况下了解并对目前的开发内容开展见习工作。



### 0930

任务名称：前端框架的学习；
上周任务：django框架的学习与实践；
本周任务：django框架的实践应用与reactjs的起步。

### 1014

任务名称：Vue3相关内容的学习；

上周任务：学习Vue3的包括组合API、数据响应的相关内容；

本周任务：继续阅读Vue3源码，并适当学习TypeScript内容。

### 1021

任务名称：Vue3相关内容的进一步学习；

上周任务：学习ES6中与Vue3相关的内容，以及基于ES6的Proxy与Reflect实现的Vue3的相关内容；

本周任务：就学习内容进行实践，参考绩效项目的示例代码练习Vue3的实现。

### 1028

任务名称：广西电子票据交付平台V1.0；

上周任务：财政电子票据交付平台的需求确认与界面布局的初步开发；

本周任务：财政电子票据交付平台的界面布局与UI效果的实现。

### 1111

任务名称：广西电子票据交付平台V1.0；

上周任务：财政电子票据交付平台的界面布局与UI效果的调整；

本周任务：前端学习并等待分配后续工作。

### 1215

任务名称：湖州日报手机H5票据平台V1.0；

上周任务：票据平台的界面布局与功能实现；

本周任务：票据平台的后续界面和功能的实现与完善；

### 1230

任务名称：实习工作的总结收尾；

上周任务：湖州日报票据平台的后续界面问题修正；

本周任务：前端技术的学习与实习工作的总结收尾；