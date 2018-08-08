---
title: Functional approach to frontend development
date: 2017-12-25 09:29:02
tags:
- functional programming
- design pattern
---

# 前端很难吗？

为什么想讲这个话题，因为咱们作为前端开发者，经常有被其他领域的工程师质疑：“你整天都在忙啥。不就是画个UI有那么难么？”当然我们可以找出很多例子来证明UI真的可以很难很高科技，比如说天猫那个魔术一般的navigation bar，但是我们加班真的在忙这些东西吗？我猜测绝大多数人跟我一样，头疼的task和bug都来自于无穷无尽的业务逻辑，表单，表格当中。所以我个人是很想赞同他们的质疑：“前端真的有那么难吗？”

很多技术都是出现的时候十分难，经过科学家和工程师们的不断努力，越来越强大，却反而越来越友好，比如说编程本身，功能已经无处不在，却早已不是那个只有尖端科学家才能学习的稀罕东西。web开发似乎是个例外，功能上的爆炸式增长的同时，javascript似乎变得越来越难以捉摸。

# 田园时代

西方的诗歌喜欢怀念田园时代，就是那个城市高楼手机汽车要啥没啥但是因为简单美好让大家觉得是遗失的天堂。古时候的Web开发也有这么一个时代，我们连JavaScript都没有，和我们提起前端脑子里“蹭”就冒出javascript可完全不一样。但我们确实可以完全不用JavaScript进行Web开发，比如经典的structs，spring，asp.net。这里我们姑且称那时为web开发的田园时代，简单，清晰，让人觉得舒服和美好。

虽然不是百分之百的场景还原，但是让我们花一分钟感受一下依赖ROR如何开发一个最基本的Web上的用户登录。用ruby是因为不用为一大堆花括号排版ppt，请大家不要关注语言谢谢。

这是很纯粹的一个MVC pattern的server side rendering,Model和Controller都是在服务器上运行的代码，View是在server端渲染好，再以html的形式返回给浏览器呈现给用户。Model的作用 ，是用来管理数据。这个例子中，我们建了一张用户表，有name和password两个字段。当然实际项目如果这么存password后果比较严重，大家不要模仿，User Model描述了我们需要校验name存在以及一个大写的变换。 View的作用，是把model用人类可交互的方式在网页上display给用户，比如绘制了一个form，并且将model中的相应字段填进去并且指定了表单提交的地址，我们在这里省略了关于样式和页面上的细节组件。 Controller用来handler business logic。这个例子里，我们只完成了认证通过的部分：将id计入session，redirect到landing url。虽然这是很业余的一种做法，只是为了demo方便，但是实际项目中也无非是用真实的持久化登陆状态的过程替换，完整的计算出一个user info并且交给landing view来渲染。(下一页)没有JavaScript，我们完成这样一个登陆的web页面。

建表(当然production中这么建表被fire了后果自负)

```SQL
CREATE TABLE user (
  id int(11) NNOT NULL auto_increment,
  name varchar(255),
  password varchar(255),
  PRIMARY KEY (id)
);
```

model

```ruby
class User < ApplicationRecord
  validates :name, presence: true

  before_create do
    self.name = login.capitalize if name.blank?
  end
end

```

view

```erb
<%= form_tag("/login", method: "post") do %>
  <%= text_field_tag(:name) %>
  <%= password_field_tag(:password) %>
  <%= submit_tag("Search") %>
<% end %>
```

```erb
<h1>Welcome</h1>
<h3>Hello: <%= user.name %></h3>
```

controller

```ruby

class LoginsController < ApplicationController
  def create
    if user = User.authenticate(params[:username], params[:password])
      session[:current_user_id] = user.id
      redirect_to landing_url
    end
  end
end

```

## UI和业务逻辑没有太多耦合

这就是我们所描述的田园时代，简单清晰。展示层，或者我们所说的View和业务逻辑，就是那段controller代码，只是通过model的字段定义有数据接口上的约定，没有其余的耦合。Controller处理业务逻辑，操作我们的model。View用来展示model。想象我们现在不用ejs作为模板引擎了，比如说用pug，controller基本不用啥改动。同样的，现在我们把db用mongodb替代mysql（事实上，我推荐个人小项目这么干），view基本不用改。即使业务变复杂，这个原则不会改变。

## 换成knockout呢。。。

```html
<div data-bind="css: { profitWarning: currentProfit() < 0, majorHighlight: isSevere }">

```

# 工业文明：欢迎来到JavaScript和SPA的世界

但是为什么我们现在都move到了JavaScript的世界，都采用SPA来开发？人们做生意都不想做赔本生意，这里也是，我们为了好处和解决问题而来。

首先，服务端渲染的performance很差。我们需要在服务端操作model，渲染出整个页面，然后以html的形式返回给前端，效果很差。哪怕计算的结果是刚才的login form下面只多了一行字：“对不起，您的密码有误”
另外，重画整个页面除了很差的perf，对用户体验的副作用也是很大的，原来focus在密码框里的光标会lose focus，重新填写还要再点一下。就像刚才Shixin讲的，这是一个ui friction
此外，没有ajax，我们无法做到异步操作。我们等待密码验证的时候什么事都干不了。
从开发者的角度，在浏览器上debug和连上服务器debug，体验不可同日而语。比如说，对于lsr，连上prod server去打断点不会是一个美好的回忆。

- Server side rendering has poor performance
- Whole page redrawing has bad user experience
- Cannot do asynchronous tasks
- Difficult to debug on servers

## 代价

就像现代文明会带来环境污染一样，我们付出了代价。

一个严重的问题，就是UI和业务逻辑开始深度耦合。现在我们不黑ko了，这是angular官网上的例子当然angular是一个优秀的框架，涉及到了和今天的topic无关的设计哲学，咱们这里就不讨论了。

```html
<div id="ctrl-as-exmpl" ng-controller="SettingsController1 as settings">
  <label>Name: <input type="text" ng-model="settings.name"/></label>
  <button ng-click="settings.greet()">greet</button><br/>
  Contact:
  <ul>
    <li ng-repeat="contact in settings.contacts">
      <select ng-model="contact.type" aria-label="Contact method" id="select_{{$index}}">
         <option>phone</option>
         <option>email</option>
      </select>
      <input type="text" ng-model="contact.value" aria-labelledby="select_{{$index}}" />
      <button ng-click="settings.clearContact(contact)">clear</button>
      <button ng-click="settings.removeContact(contact)" aria-label="Remove">X</button>
    </li>
    <li><button ng-click="settings.addContact()">add</button></li>
 </ul>
</div>
```

### debug: 更难诊断

这样的第一个后果，出现在deBug上。确实，用浏览器比起连上server，打断点的体验好一些。但是打几个断点？在哪打断点？传统的MVC，debug很难，但是，找到rootcause很简单。刚才的例子啊，名字变到hello之前了，那我们知道在view里出问题了。发现那么不是我们想要的，ok看下controll里findUserById的调用处打断点。现在混合之后，页面上的元素组织甚至在和model或者viewmodel里数据的拓扑结构绑定，我们无从查起。更不用说，所有data binding为基础的框架，我们更多的看到这样的error message所以，来到浏览器上，我们开始debug不难了，但是，还是得加班

### 扩展：难以refactor和修改设计

第二个问题，就是我们的修改，变麻烦了。讲这样一个故事啊，我们的campaign creation，也就是让advertiser创建campaign流程中，需要用户设置自己campaign的budget。这个要求即使对于初学者也是小菜一碟。

我们的campaign creation分为4个大的步骤。PM想到了，他在中间步骤的时候会重新考量自己的业务，这样在最后一步，他经常发现在第一步设置的budget需要修改。当然，依然还有很多advertiser对于自己有多少钱是心中有数的，所以想在第一步设置好。所以，我们解决ui friction的办法就是，两边都放上。

对于以前的模式，这不是问题，在最后一步view里面加一个budget input，把model填进去，搞定。如果你在用什么java ant tomcat，那就敲一下f5然后出去吃夜宵。

现在的话，嗯，比方这样一个问题啊：

如果我们让这两个budget control是一个instance:因为每一步的next和back button都会validate当前step所有的form,那么实际上，第一步和第四步的都执行了自己步骤以外的validation，这导致了很多bug；如果让他们是不同的instance，需要构造两次不说，其他受budget影响的component都需要监听或者分辨来自不同budget control的事件，或者判断传入的instance是不是当前想要的。数据和validation同步也是可能导致问题的。

我们这个讨论还没结束呢，另一个crew开始找我们集成shared budget。业务上来说，几个campaign可以用同一个shared budget；工程上来说，就是我们原来写的budget的control成为了新的budget panel的子组件。在viewmodel里，hierarchy同时影响了页面上的布局和逻辑里的关系，牵一发而动全身。

另外一方面，如果将来我们有更多的选项，需要用dropdown替换radio button，显然这种UI和业务逻辑耦合的做法会带来更多的困扰

### 新问题：引入异步带来的坑

再就是concurency。新加入的功能引来了新的问题。

我们的render和destroy方法都是同步的。因为lazy loading的需求，我们需要在render的时候再加载一个第三方库，加载成功之后会创建一系列新的实体在其子节点上。destroy的时候，需要讲他们一一销毁：这时，一个很容易出bug的问题诞生了：你怎么知道destroy的时候render及其异步操作已经全部完成。稍不小心就会让程序进入无法预测的异常状态。

1. 改框架，render和destroy都支持promise。
    - 好主意，加油（微笑）争取明年release（doge）
2. 加全局flag
    - 我们确实很多时候是怎么干的。
    - 异步之后又fork出新的异步task怎么办
    - 你怎么能保证线程安全？怎么能保证flag不被修改？如果之后的业务告诉你确实要修改flag怎么办？比如说，有两个同时进行的lazy loading。掀桌。

# 文艺复兴

我们付出的这些代价，就是可以讨论的可以改进的空间。我们想变得更好。原来简单，清晰。我们要重新变简单，变清晰。

那么，当时move到JavaScript SPA的时候，不就是把server sider rendering换成client side rendering嘛，其他的逻辑组织一切都不变，完全平移到client side。或者说用javascript强行写一个ror或者asp.net搁浏览器里跑，可行吗？

可行倒是可行，但是频繁的重画整个dom tree，依然让perf和体验不理想，可以归结为ui friction。

那为什么现在我们又能重新讨论这个话题了呢？上周关于react的分享中，我们知道了virtual dom这个概念。这个想法太赞了，我们在code里不管怎样重新render都没有关系，多了一层来帮我们计算delta来作partial rendering，即使我们让数据一圈一圈转起来，都不用担心perf 都不用担心ux了。
另一方面，现在的UI，组件话已经越来愈好，对于很多局部的小控件，其实啊，重画是完全可以接受的，并不是ui friction，比方说，你就是一个小小的date picker之类的。
好了，现在我们可以回头找一下那一些被迫放弃的田园时代的美好吧。不过千万不要忘了，异步的问题是新带来的，我们还需要解决

## 如何思考？

既然已经打出了文艺复兴的旗号啊，接下来咱们就会在把那些以前落了灰的东西翻出来，收拾收拾里面精华的思想，然后总结一套 让我们的开发重新变简单 的思路和方法。

实际上react分享的时候，已经有这样的问题被提出来，如果我把view model中引起问题的那个数据流方向干掉，是不是就可以实现跟react类似的功能。perf问题？没关系，也有很好的开源的virtual dom技术。

why not？没错，我们今天的topic，不需要任何新的framework也好 library也好。当然，有些跟我熟一点的知道我是某些新东西的粉丝。放心，我不是来传销他们的。如果你有听过启发我讲这个topic的那些新东西，希望你能理解 原来是这个原因 我才喜欢他 如果你不知道那是啥 更好 希望你能知道 啊 原来老的代码里 还能这样想。

### Business requirements -> mathematic problems -> code

我们上班是来干嘛的？

我们是来解决问题的 然后用解决问题的能力兑换成薪水

程序员怎么解决问题？

我们把实际的业务抽象成数学问题 然后把解决这个数学问题的方案用代码写出来

## 从田园controller开始

重温一下什么是controller：它是用来handle业务逻辑的，具体的，干了两件事：第一，操作model，用时髦的说法，这句话很重要：执行model的状态转移。第二，根据业务，把相应的model交给相应的view，进行渲染

### 我们的业务逻辑

- 描述model的状态转移：实际上就是根据一些条件，对model执行了一个函数，得到了一个新的model
- 重复一遍：执行了一个函数，得到了一个新的model：而不是执行了一条命令，修改了model。
- 为了让问题更简单，我们做一个尝试，让刚才说的“一些条件”全部由操作时的parameter来表示，其他因素要么放进去，要么被忽略。
- 我们还想要更进一步的简单和清晰。OK，model里有字段，还有描述model变换的方法。我们不要，我们只需要一个最最简单的数据结构，比如说，简单到就是一个简单的plain object。我们给它一个更准确描述这个plain object的名字： state
- 对于parameters 推广到普遍适用的情况，我们给他的名字叫 action，动作，来自哪里的动作？我不关心

```
state_next  =  r(state, actions)
```

### 我们的UI

再来看controller的另一个任务：把相应的model交给相应的view，进行渲染。

- What is a view?
  - To display model

也就是说，View是什么？View就是把model扔在屏幕上，使其可以和用户交互的一种映射。回忆一下高中数学，把model可能的状态看成定义域，UI所有可能的显示看成值域，这是什么？一个函数

跟之前一样的解释：model拓广到states 我们得到了

```
UI = f(state)
```

是的 UI只听state的

你想说 我的页面，要在晚上10：00到早上8：00把页面弄成适合夜间的尿黄色。OK，请在state里描述时间。
我的页面，在中国用高德map，在美国用google map。OK，请在state里描述location。

UI就干你改干的事，清清楚楚。

state怎么这么累？f说，我不知道，你问r去

这就是今天我想阐述的核心

**敲黑板重点**

```
UI = f(r(state, action))
```

# UI = f(r(state, action)): 欢迎来到函数的世界

## 我们的工作

- 怎么描述一个时间点上静态的状态？这是state的问题
- 怎么描述发生的一切和其他实体，比如用户的交互？这是action
- 怎么让state和action相互作用，完成业务逻辑的要求？找一个合适的r。我会忍住不说我最喜欢的那个很新的r，我后面会证明，你们正在用的那些被嫌弃的老技术，也可以是很好的r
- 怎么在真正的页面上让state说人话？找一个合适的f，同样的，别看不起你们现在用的。

更进一步解释我给出的这个式子。

第一件事情：前面已有说明，f和r就是数学意义上真正的函数，不是我们平时干的写作function用成command或这procedure的那些假冒伪劣产品。在编程上 我们称为纯函数。
第二件事情：只是再强调一遍，我不是来推销某个framework的，这种pattern，对所有的中立。
第三件事情：这种pattern，实际上已经解决了开始提出的一个重要的问题：UI和business logic耦合

## 我们的数学基础

### 范畴论 Category theory

首先是范畴论：将这些概念形式化成一组组的“物件”及“态射”

- 解释了为什么pure function对干扰免疫，没有副作用
- 思考问题的时候，一定注意把pure function理解成pipe，而不是execute，重点！pipe！

这就是函数式编程最基本的模型：函数就是箭头，值就是点（后面λ演算可以更丧（gan）心（de）病（piao）狂（liang）的指出值也是函数），除了两个点，这就是一个纯粹的数学模型，不会和任何其他的点或者箭头发生交互。
这个箭头不会吃掉它出发的点，不会改变它到达的点，它就是像一跟管道一样。
没错，一定要把函数理解成管道，不要理解成命令！我们的思维在这里转变。

### λ演算 λ calculus

λ-terms（解释了为什么“函数是一等公民”）形象的说，就解释了函数，不仅是箭头，也可以是点。函数本身就是一种值的类型。
evaluation（解释了柯里化的数学原理，解释了FP对于异步和并行的优势，启发我们不要用传统的考虑“值”的思路（call by value）去理解而是call by name）
λ表达式只允许一个参数，柯里化可多个。
函数应用可以把函数整体代入，不是先要求值：这就是我们天天写的callback啊

## 我们的工具

**纯函数 pure functions**

重温一下pure funciton的概念，之前react的分享里已经提到过的

两个要求：第一：输入确定，则输出确定。第二：没有副作用，或者说，它只能干一件事请，就是输出一个值，其他什么改一个全局变量啊全都不行。

换句话说，pure function就是纯数学意义上的函数，编程上是stateless function，没有internal state。

### 我们建造新大厦的砖块

在这个pattern中，pure function就是我们编程的砖块。为什么选择这个当砖块？因为它最简单，也最容易复用。

首先，它完全对这样一大类bug免疫，就是一堆人在改同一个可变的东西，什么out of sync或者inconsistent的问题。因为每块砖只关心自己的输入。
另外因为这个特性，它一定是并行编程的很好的候选方案。
第三，它完全不关注自己受谁的影响，自己会影响谁。在哪都一样，很容易的就搬来搬去。

我猜有人会用随机数生成的问题来challenge。没问题，只需要知道随机数需要seed就好了。

当然，等真的写code的时候，我们想的一定是怎么解决实际问题，而不是怎么套用design pattern，我们想的是怎么用pure function的思想提高我们思考的效率，而不是绞尽脑汁套pure function来降低我们思考的效率，be smart

> Javascript并不是一门非常适合函数式编程的语言

## 我们的动机

- We want UI to be predicable
- We want states transforming be predicable
- Reusability, Flexibility and Extensibility

## 我们的做法

### Immutable

不推荐的

```javascript
function selectionProjectionHandler(state, config, model) {
  if (config.selectable) {
    // ...
  } else {
    state.errorCode = 'error_unselectable’;
  }
}
```

推荐的

```javascript
function selectionProjectionHandler(state, config, model) {
  if (config.selectable) {
    return /* compose new state with item selection */
  }
  // Immutable
  return Object.assign({}, state, {
    errorCode: 'error_unselectable’,
  });
}

```

其实很简单 别改 返回一个新的

当然这个例子说明了一个问题是 如果你的目标是学习函数式编程 显然JavaScript不是一个合适的选择 这里的语法看起来不太直观 甚至可以说 是一个work around： 因为那个新的对象 其实就是这个空的花括号修饰出来的 没错 这不是immutable 我们产生了修改这个空object的副作用 强调一遍 我想讲的是怎么借助这些思想让我们开发变简单

而immutablejs这个库的作用 就是强制copy on write 对于map array set这些基础的结构做了封装

recap一个pure function： 干且仅干一件事情 接受一个输入 给一个输出


### Partial applying

偏函数解决这样的问题：如果我们有函数是多个参数的，我们希望能固定其中某几个参数的值

```javascript
function log(status, context, message) {
  // fancy logging code here
}
function info(context, message) {
  return log('info', context, message);
}
function dbInfo(message) {
  return info('db', message);
}

```

### Currying 柯里化

下面来讲柯里化 因为λ演算表现成code 最纯净的函数定义是一个参数进去 一个参数出来 但是我们实际业务很容易描述成具有多个参数的情形 所以我们需要柯里化 只提供一个参数 其他的参数作为这个函数的“环境”来创建，来回归原始

大家回想一下自己备战高考的岁月啊 数学里函数的部分占了大头对吧 大家重温一下只有一个参数的题的难度 和有多个参数的题的难度 应该就有一个感性的认识了 现在我们做项目 不就是两步：先把实际问题数学化 再把数学解答代码化？好 想展开一个话题 随便google一篇文章都比我这讲得好 就不浪费时间了

柯里化是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。指的是将一个具有多个参数的函数，转换成能够通过一系列的函数链式调用，其中每一个函数都只有一个参数

我们还是拿log这个例子来说明

```javascript
function log(status) {
  return function(context) {
    return function(message) {
      // fancy logging code here
    };
  };
}
const info = log('info’);
info('db-context')('message');
const dbInfo = info('db'); // create a new function again
dbInfo('something happened’);
log('other level')('some context')('a message');

```

这就是柯里化 其实没那么玄乎 我们把一个关注三个参数的问题 变成了三个分别只有一个参数的函数的链式调用

我们想要创建之前的info函数 就把status=info理解成创建这个新函数需要的环境

请看 这很生动的说明了函数作为一等公民 充当返回值这个角色 现在我们想打一条log 就是调用这个新函数

或者说 我们还想把dbInfo这个新函数创建出来 道理也是一样的 不过这里我们又可以强调一下了 一定要是纯函数 如果不是 比如说dbInfo调用的时候有可能有对info甚至log产生副作用 这整个体系就崩塌了


可以说 柯里化就是这样一个神奇的工具 我们几乎可以只依靠它 又简洁又优雅的的解决特性与共性的问题 给予我们代码良好的复用性 灵活性 扩展性 理解起来 我们只需要牢牢记住 函数是一等公民

### Function compose

我们来复习高中知识 啥叫函数组合？很简单 f(x)是函数 那么 f(g(x))就是函数组合

```
UI = f(r(state, action))
```

```javascript
const compose = function(f, g) {
  return function(x) {
    return f(g(x));
  };
};
```

## 我们的收获

1. Reasonable

很多人相信使用纯函数最大的好处是引用透明性（referential transparency）。如果一段代码可以替换成它执行所得的结果，而且是在不改变整个程序行为的前提下替换的，那么我们就说这段代码是引用透明的。
由于纯函数总是能够根据相同的输入返回相同的输出，所以它们就能够保证总是返回同一个结果，这也就保证了引用透明性。

2. Cacheable

首先，纯函数总能够根据输入来做缓存。实现缓存的一种典型方式是 memoize 技术

- 值得注意的一点是，可以通过延迟执行的方式把不纯的函数转换为纯函数：

```javascript
var pureHttpCall = memoize(function(url, params){
  return function() { return $.getJSON(url, params); }
});
```

3. Portable / Self-Documenting

纯函数是完全自给自足的，它需要的所有东西都能轻易获得。

在 JavaScript 的设定中，可移植性可以意味着把函数序列化（serializing）并通过 socket 发送。也可以意味着代码能够在 web workers 中运行。总之，可移植性是一个非常强大的特性。
命令式编程中“典型”的方法和过程都深深地根植于它们所在的环境中，通过状态、依赖和有效作用（available effects）达成；纯函数与此相反，它与环境无关，只要我们愿意，可以在任何地方运行它。
你上一次把某个类方法拷贝到新的应用中是什么时候？我最喜欢的名言之一是 Erlang 语言的作者 Joe Armstrong 说的这句话：“面向对象语言的问题是，它们永远都要随身携带那些隐式的环境。你只需要一个香蕉，但却得到一个拿着香蕉的大猩猩...以及整个丛林”。

4. Testable

纯函数让测试更加容易。我们不需要伪造一个“真实的”支付网关，或者每一次测试之前都要配置、之后都要断言状态（assert the state）。只需简单地给函数一个输入，然后断言输出就好了。

5. Parallel

最后一点，也是决定性的一点：我们可以并行运行任意纯函数。因为纯函数根本不需要访问共享的内存，而且根据其定义，纯函数也不会因副作用而进入竞争态（race condition）

# One more thing

## Flux pattern

- View: representation layer. Controller views.
- Action: raise actions from representational layer.
- Dispatcher: receive actions and invoke callbacks.
- Store: respond to dispatched actions.

## Views controllers pattern

- Separate controllers(container components) and views(representational components)

# 几个肯定会被问到的问题：

**Q: In big team co-work, how to break projects down?**

(如果我们用函数式思考，一切非常自然)

回答这个问题，我建议大家先来这样思考我们的项目，想象成一个表格，或者数据库表，也就是一个二维的矩阵，一个维度是不同的业务实体，在这里就是不同种类的campaign；另一个维度与它垂直，就像是auth, i18n, theming，routing等等，你每一行代码的作用，就都可以用一个向量来表示：我这一行，是处理DSA的i18n。那么两种思考问题的方式其实就是一个地方不一样：把哪个维度看成行，把哪个维度看成列。

传统的方法是根据业务来思考的。而这里推荐的另一个看问题的角度，是通过可扩展性，可维护性，（说人话：呼应标题，让前端开发变简单）的方面来思考。这和数据库很类似：行扩展很简单，一条SQL语句即可，列扩展相对麻烦。所以我们用middleware来描述我们的列:
一方面，借助柯里化，当我们需要扩展列（增加新的middleware）时，代价不那么大
另一方面，当我们增加业务实体时（这是我们的主要工作，除了架构师）进行简单的行扩展

比方说 auth, i18n, theming，routing等等作为middleware避免了大量重复代码。我们平时在处理这些逻辑的时候，就算不是copy paste，互相之间的差别也不大，基本上是配置上的差别。

那么我们会问这样一个问题，如果我要在我们的项目中加入一个新的entity，举个例子，我们已经有了几种campaign，search campaign，shopping campaign等等，现在我们又要加入DSA campaign.

传统的做法是加入一个DSACampaignController,里面包含CRUD的actions，然后创建相应的View，templates，和campaignMT，adInsight MT integrate之后release，当然这些和之前其他campaign的code会有很多重复但是我们似乎也能接受（其实是习惯了）

按照上面的做法，是不是说我要在auth里添加一点DSA的配置，i18n里添加几个DSA的规则，说起来不用写完整的重复的controller啊之类的，但是这种散弹式的修改真的不是团队协作的毒药吗？怎么不像上面说的“行扩展”那样简单啊？
回答：
1. 设计出声明式编程的接口很难吗？设计出集中配置项的语法糖很难吗？最最简单的，有一个middleware来派送config给不同的middleware总能做到吧。
2. flux pattern中一个重要的角色是dispatcher，具体的应用中不同框架有不同的设计，但是功能都是：把不同的action分发到不同的store或者store里的不同字段。比方说一个简单的设计，不就是hash map吗？添加新的DSA campaign type，可以设计成为为这个新的entity action添加新的reducer（就是前面讲到的r的组件，或新的一组reducer）

当然由此还会有一个问题：如果维度不够用怎么办？拿业务上说就是：我们的business entity不一定是扁平的结构，就比如说，我们有不同type的campaign，这是一层，不同type的adgroup，ads， keywords等等，然后campaign,ads,keywords这些又是拿到一层考虑：

flatten：可能很多时候我们并不需要关系这些业务上的树形关系，共性和特性就是柯里化中的小技巧了。
数学上的分形，或者结合micro service的理念，通过子APP的方式来完成。

可能还是有点抽象，还是举projection grid中的例子吧：

不同场景下，datasource的不同，toolbar构造的不同，只体现在配置不同上。
功能上的扩展，对于开发者来说，考虑两件事情（1）在哪里插入projection（2）插入什么样的projection 比如说，我们在product groups 中要用table的rows来体现业务上的树状层级，我们就加入一个tree-plugin（与product groups业务本身无关），我们考虑的是：
在哪里pipe：修改structure projection pipe啥：加入parent node row
在哪里pipe：修改content projection pipe啥：添加icon和缩进描述层级关系

**How to compose async actions?**

我只想回答这么一句话

```
f_async(…args)  ≡  f(condition, …args)
```

**How to share components to public?**
- We share widgets and controls. we should not share business.
- We can share universal helper functions in typical pure function way.
- Pure functions are even easier to share: No side effects. Predicable





