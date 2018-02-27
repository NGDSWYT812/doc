#梳理mvp之model篇


日期：2017年9月12日</br>
作者：王宇滔

本次分享主要梳理下mvp的相关知识，详细介绍下model的知识以及model引起的思考。

##什么是mvp
MVP，全称 Model-View-Presenter。而mvp的鼻祖是mvc，mvp是从mvc演化而来的，所以他们有着相似的地方：Controller/Presenter负责逻辑的处理，Model提供数据，View负责显示（mvp和mvc最大的不同是model与view能否是完全隔离开的）。而对mvp对android开发而言，与传统的开发最大的不同就是mvp把activity的职能进行了拆分。个人认为mvp只是一个开发的模式和套路，在一定的开发场景下可以提高开发效率。

而mvp大致有这两种套路：

* Passive View（pv）被动视图模式
* Supervising Controller（soc）监听控制器模式

见demo

优点：model，presenter的复用；代码结构清晰；</br>
缺点：类变多，开发繁琐，整条业务逻辑线分散；

##M（Model）
Model：主要是数据模型和业务逻辑。向p层提供数据接口。

##V（View）
View：将数据呈现给用户。一般的视图包含用户界面(UI)，而不包含业务逻辑。

##P（Presenter）
P:各种控制器。一般负责调控view和model之间的协作。

##Model详解
简单的model可能看起来像是一些set，table之类的东西，但真正的model不是指这些。mvp中所指的model代表着一类组件(components)或类(class)，这些组件或类可以向外部提供数据，同时也能从外部获取数据并将这些数据存储在某个地方。简单的理解，可以把模型想象成“外观类”，对于外部调用者presenter，能看到的只是提供数据的一些接口。

###model主要负责

* 从网络，数据库，文件，传感器，第三方等数据源读写数据。

* 对外部的数据类型进行解析转换为APP内部数据交由上层处理。

* 对数据的临时存储,管理，协调上层数据请求。

###model的大概的结构
interface(向外提供数据的接口)</br>
data(各种数据)</br>
adapter(解析适配数据)</br>
source(网络，数据库，文件，各种数据提供者)</br>

###model中的实体类
说到model就会想到各种实体类：

* POJO
全称为：Plain Ordinary Java Object，即简单普通的java对象。一般用在数据层映射到数据库表的类，类的属性与表字段一一对应。

* PO
全称为：Persistant Object，即持久化对象。可以理解为数据库中的一条数据即一个BO对象，也可以理解为POJO经过持久化后的对象。

* BO
全称为：Business Object，即业务对象。当某个业务比较复杂，用到比较多的业务对象，这时候一个一个传递业务对象是很麻烦的，于是乎就有了bo。

* **DAO**
全称为：Data Access Object，即数据访问对象。就是一般所说的DAO层，用于连接数据库与外层之间的桥梁，并且持久化数据层对象。DAO其实是来源于J2EE的一个设计模式，当初的目的也是使得企业更换数据库时，不用影响模型层的代码。目的有点类似于图片加载框架的上层封装。

* **DTO**
全称为：Data Transfer Object，即数据传输对象。对后端来讲DTO就是包装来给出去的对象。对客户端来讲，就是数据适配器解析出来后对应的完整对象。这些对象用于向数据层外围提供所需的数据。

* **VO**
全称为：Value Object，有的也称为View Object，即值对象或页面对象。一般用于其他层向view层封装并提供需要展现的数据。

###关于model层的一些思考
项目中只有dto or vo？ dto == vo ？</br>
我遇到过的许多项目，dto对象和vo对象是没有明显的分层的；
因此你是不是也遇到过如下几个问题？

* 当使用model取得数据时，你是不是会担心这个对象或者对象的某个字段是null，从而逻辑代码中一堆的空判断？
* 接口返回的数据结构或字段突然变更了，丫的我要修改view中的set代码(什么int改string啊，string改成int啦，代码里面一堆format)。
* 接口返回的数据结构不是我想要的啊，要么去求服务端能不能给我一个我想要的数据结构啊，要么自己去做一堆转化(日期format，多类型列表构造)。


解决方案就是dto和vo对象的分离，在dto对象中做一个转化(没实践过，拿出来跟大家讨论下，看起来很美好，但是是否真正符合我们当前项目？)。</br>
[Android-CleanArchitecture](https://github.com/android10/Android-CleanArchitecture)

#预告
* presenter详解
* view详解
* 用mvp思想建造的加载器DataLoader 


