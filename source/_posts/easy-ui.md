---
title: Make UI development easy again
date: 2017-12-11 10:26:12
tags:
---

作为前端开发者经常有被其他领域的工程师质疑：“你整天都在忙啥。不就是写个UI有那么难么？”当然我们可以找出很多例子来证明UI真的可以很难很高科技，比如说amazon那个魔术一般的wunderbar，但是我们加班真的在忙这些东西吗？我猜测绝大多数人跟我一样，头疼的task和bug都来自于无穷无尽的entity CRUD，表单，表格当中。所以我个人是很想赞同他们的质疑：“前端真的有那么难吗？”

# It was easy

若干年之前，我们并没有javascript，也没有SPA，那时的web应用开发是非常简单清晰的。我们以经典的ROR（ruby on rails, 很多被广泛的使用的框架，如structs，spring, Yii, asp.net都深受其影响）为例：

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

这一切很简单，在经典的MVC+serverside rendering的架构中，controller和model都是在server side运行,controller来管理business logic：调用哪个model（User），进行什么操作（authenticate），结果是什么（成功则跳转），model用来管理数据：User的字段,query（本例中映射到数据库）等操作。而view，很诚实的根据model里的数据来渲染页面，提交表单。我们没有JavaScript，发不出change，onClick事件，没有办法在view里面进行正则验证，这些逻辑都由controller完成。

*Recap*：

1. UI = f(model),不会受其他data binding或者events的影响
2. view和controller之间划分了一条明显的界限

好处：
1. 简单清晰，开发效率高，学习成本低
2. 想象我们现在不用ejs作为模板引擎了，比如说用pug，controller基本不用啥改动。同样的，现在我们把db用mongodb替代mysql（事实上，我推荐个人小项目这么干），view基本不用改。

你换angular试试？把validate写进filter里:P


举一个现实的例子，在campaign ui从ui server migrate到SPA的过程中，我们在UI server端最费心的是什么？删掉这个controller会不会有request 404。实际上，如果不是这么大规模的替换，根本用不了这么担心，比如上面提到的换模板引擎或者换db已经够大了吧。不用说简单（呵呵）的修改逻辑或者修改用户体验。

换knockout呢？看不懂的console error等着你


## why we move away

To reduce UI friction!!

1. 每次重新渲染页面的perf无法接受
2. 复杂交互的需求：animation等

拿这写个gmail？微软的卧底才这么干。还是用angular吧=.=

## why we want it again?

现在前端有大量MVC，MVVM框架，他们很优秀解决了无数的问题，但同时也引起开篇提出的质疑：现在前端是加班重灾区（jquery要接大锅，但与本文主题无关，略，并且相对于功劳，它不优雅之处并非主要）立下汗马功劳的events机制和data binding反过来在bite我们：

1. hard bugs

  在campaign creation的第一步和第四步，我们有两次让用户set buget的机会。如果我们让这两个budget control是一个instance:因为每一步的next和back button都会validate当前step所有的form,那么实际上，第一步和第四步的都执行了自己步骤以外的validation，这导致了很多bug；如果让他们是不同的instance，需要构造两次不说，其他受budget影响的component都需要监听或者分辨来自不同budget control的事件，或者判断传入的instance是不是当前想要的。数据和validation同步也是可能导致问题的。

2. DCR and rebranding

  我们做了决定，deprecate monthly budget。然而 有很多已经存在的campaign apply了monthly budget， 所以settings页面需要保留monthly选项，creation页面需要删掉。

  当我们已经在pilot 紧张的准备GA的时候，另外一个feature也要开始准备release了：shared budget。原来的budget control成为了一个child 和shared budget的control一起组成新的budget panel：所有的事件和接口，都需要转发。

3. painful debugging experience

  我们有很多别的限制：不同的location，不同的language，budget的currency和limitation不一样，同时，bid是受到budget的限制的，traffic estimation是收到budget影响的。
  别忘了，我们有两个budget control，而且未来有可能，面包屑上方的header里，会出现第三个。
  别忘了，shared budget是和别的campaign关联的。budget除了value，还有type。是的，type还会变。你可以想象到我们在调试其中关于validation,关于filter，关于format，等等，bug的时候，不知道root cause在哪一脸懵逼的表情吗。

4. async

  我们的render和destroy方法都是同步的。因为lazy loading的需求，我们需要在render的时候再加载一个第三方库，加载成功之后会创建一系列新的实体在其子节点上。destroy的时候，需要讲他们一一销毁：这时，一个很容易出bug的问题诞生了：你怎么知道destroy的时候render及其异步操作已经全部完成。稍不小心就会让程序进入无法预测的异常状态。

  （1）改框架，render和destroy都支持promise。
      - 好主意，加油（微笑）争取明年release（doge）
  （2）加全局flag
      - 我们确实很多时候是怎么干的。
      - 异步之后又fork出新的异步task怎么办
      - 你怎么能保证线程安全？怎么能保证flag不被修改？如果之后的业务告诉你确实要修改flag怎么办？比如说，有两个同时进行的lazy loading。掀桌。

哦，对了，我们GA了之后，另外一个crew开始在我们的基础上做multiple language。原来language由enum，成为了数组或者enum。嗯。。。
我们加班就是在干这样的事情的。

# Make it easy again

## We want both

回想一下我们为什么放弃了那些田园时代美好的东西：1. perf 2. ux requirements

不就是把server sider rendering换成client side rendering嘛，其他的逻辑组织一切都不变，完全平移到client side。或者说用javascript强行写一个ror或者asp.net搁浏览器里跑，可行吗？

当然可行，不过频繁的dom操作，依然让perf和体验不理想，可以归结为ui friction。好吧，我们写angular吧，我们写knockout吧，我们写backbone吧。

喷了一大圈，那些大神们果然还是比我们厉害啊。

### Not a blocker now

没想到现在我们迎来了一个大救星：virtual dom。放心的render，rerender吧，virtual dom刚我们干了reduce ui friction的活儿。咱们放心前进。

另一方面，对于很多局部的小控件，其实啊，重画是完全可以接受的，并不是ui friction，比方说，你就是一个小小的date picker之类的。

好了，现在我们可以回头找一下那一些被迫放弃的田园时代的美好吧。、

*Recap*：

1. UI = f(model),不会受其他data binding或者events的影响
2. view和controller之间划分了一条明显的界限

## How？

### 状态递归

controller是啥？描述状态转移的。compose出新的model 把这个新的model，交给新的view

**controller: old_model -> new_model**

根据什么计算？废话，传进来的参数啊。别的东西我看不见看不见。

```
new_model = r(old_model, params)
```

在这里，我们并不像局限于某一种特定的pattern来讨论这些问题：viewmodel还是model，who cares？我们可以都叫他state

params也太局限了，我们也不想局限于某一种代码组织方式，controller function？controller class？这不重要，我们是来推销哲学的。我们就叫它action吧。

敲黑板，划重点：

```
state_next = r(state, action)
```

r is a pure function!r is a pure function!r is a pure function!

### UI = f(state)

和以上同样的想法，把model扩展成state状态得到。

```
UI = f(state)
```
f is a pure function!f is a pure function!f is a pure function!

### pure function

1. what

- Given the same input, will always return the same output. 给定输入，则输出给定。
- Produces no side effects. 没有副作用。

The simplest reusable building blocks of code in a program
- Immune to entire classes of bugs that have to do with shared mutable state
- Great candidates for parallel processing
- Easy to move around

*FP(functional programing)的数学背景(如果时间允许且听众感兴趣的话)*

(1) Category theory

(解释了为什么pure function对干扰免疫，没有副作用)
(思考问题的时候，一定注意把pure function理解成pipe，而不是execute，重点！pipe！)

- category: an algebraic structure that comprises "objects" that are linked by "arrows“
- Consider the “category” as “container” which contains:
  - values
  - transforming of values

(2) λ calc(lambda演算 然而JavaScript里的lambda 表达式不是真lambda表达式)

- λ-terms（解释了为什么“函数是一等公民”）
  - variables: x
  - function creation/abstraction: λx.E
  - function application: E1 E2
- evaluation（解释了柯里化的数学原理，解释了FP对于异步和并行的优势，启发我们不要用传统的考虑“值”的思路（call by value）去理解而是call by name）
  - Alpha equivalence( or conversion )
  - Beta reduction
    - call by value
    - call by name

2. why
- we want UI to be predicable
- we want UI transforming be predicable

(不要拿随机数生成来抬杠，因为随机数的生成，是需要seed的)

3. benefits

- Easy to control the order
- Easy to pass in additional data
- Easy to make reusable reduce functions

4. How

- currying

```javascript
function log(status) {
  return function(context) {
    return function(message) {
      // fancy logging code here
    };
  };
}

var info = log('info'); // create a new function "info" by fixing first argument
info('db-context')('message');

log('other level')('some context')('a message'); // use the original function directly with all arguments

var dbInfo = info('db'); // create a new function again
dbInfo('something happened');

```

- partial

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

- immutable

  - Some side effects avoided.
  - Pure functions easier to write.
  - Fast change detection.
  - Immutable data can be safely cached.
  - Easier to implement undo.
  - Concurrency.

### Deep in r - centralized state management

1. state is read-only

  - It is immutability
  - all changes are centralized and happen one by one in a strict order
  - Easy to be logged, serialized, stored, and later replayed for debugging or testing purposes.

2. Changes are made with pure functions

### Deep in f - UI as function of state

1. One-way data binding
2. single source of truth

	我们之前遇到的各种前端框架显然不满足f(Status) = UI，通常来说，应该是
	  f(Status, 用户交互，推送信息，其他UI联动变化，……) = UI
	那么React必然要做的事情就是实现一个:g(上面其他因素) = status，这就是需要单向数据流的原因。
	我个人的体会，因为可能已经习惯了诸如knockout、angularjs等成熟的框架提供的各种完善的two-way data-binding的机制，第一眼很难接受React的one-way data flow，而且写一些很小的component的时候（比如hello world之类的），会发现很简单的事情反而需要我们花一些额外的精力。
	比如一个最简单的input框，如果用ko，只需要把他和vm上一个observable绑定就好了，在ng中也是很简单的ng-model=xxx，但是突然React告诉你，不行，你要绕一圈，你的输入只能作用于status的update，绕一圈才能回到UI的变化，当然不习惯。
	Facebook的工程师们肯定不是吃饱了撑的才这样设计。我们可以想象一些稍微复杂的逻辑——比如，如果这个input框需要把用户的输入全部转成大写字母，或者拒绝掉一些非法字符。当然，现在的框架已经足够优秀，我们可以pass in各种custom filter。问题就在于当这些需求一个一个堆砌起来，逻辑就会逐渐变得混乱，filter，suscription，events等等聚集在一起，代码的可读性会严重降低且难以维护。当UI中的元素越来越多，业务越来越复杂，这些不便利之处会指数级爆炸。
而单向的数据流就不存在这些问题了，flux中有一个很好的Dispature和Store的概念，就像现代物流的集散分拣中心一样的（此处谢绝抬杠），一件快递的时候看起来不必要，但是成千上万的数据涌来，他依然能把所有的东西处理得井井有条，所有的东西就像阅兵式上的方阵，清晰有条理。我们会被从filter，suscription，events中被拯救出来，只有两样简单的事情：status，UI。

# Tail

我们不是来sell某一种框架或者技术的。更多的是想抛砖引玉，启发大家如何借助一些最新的力量，在保留我们已有的财富的同时，找到一次前端的“文艺复新”。当然和文艺复新一样，这里也是旧瓶装新酒。这里的flux pattern和传统的MVC相比，也有了不少变化，下面会说明。

可能喜欢在技术上赶时髦的同学已经想把那几个框架或者技术的名字呼之欲出了。但是这不是这里想要deliver的东西。这里只是讲一种pattern，我们更希望它随时随地，哪怕我们在处理没有用那些新的框架的时候，也能启发我们，让我们的工作更加简单。用到的时候，更能体会作者的良苦用心，为什么这样做，而不是API的搬运工。事实上，这种思考问题的方法，这种设计的思路，真的不需要通过哪个框架或者技术才能实现。Jquery？Why not？

## case study: projection-grid

1. predicable: 只要projection确定了，内容就确定。
2. renderer并不关注projection如何被pipe，如何被compose，只需要关心render model长什么样子就能画出grid
3. React，vue，backbone版本只是renderer不一样，如何计算render model完全不用重复。所以：不同情形下可以重用同一份plugin！
4. Debug：只需要关注render model，就能查找是compose model发生的错误还是在render时发生。
5. Test：对于core，只需要关注IO，对于renderer，只需要关注给定render model之后的snapshot。
6. Extensibility: 通过pipe更多的plugin。plugin是pure function或plain object(这是语法糖，实际上我们都处理成reducer，即，根据现有配置和参数（state and action）返回新的配置（new state）)

## Flux pattern

- View: representation layer. Controller views.
- Action: raise actions from representational layer.
- Dispatcher: receive actions and invoke callbacks.
- Store: respond to dispatched actions.

### Controller Views Pattern

- Separate controllers(container components) and views(representational components)
- Benefits
  - More portable.
  - Less cognitive overhead.
  - Easier testing.

## Frequende asked questions:

Q: how to share components to public?
A:
- We share widgets and controls. we should not share business.
- We can share universal helper functions in typical pure function way.
- Pure functions are even easier to share: No side effects. Predicable
- Higher ordered components.

Q: how to compose async actions?
A:
这不就是λx condition的问题吗？柯里化。看似复杂的问题背后一定有简单的数学原理。

Q: In big team co-work, how to break projects down?
A
- Trandional: application -> modules -> controllers + views + models(viewmodels) | MVVM or other patterns
- Applying middleware:(如果我们用函数式思考，一切非常自然)
  - application = middleware1(middleware2(middleware3(...midlewareN(config))))
  - or application = middleware1(config1)(middleware2(config2))(...)(middlewareN(configN))

(如果我们用函数式思考，一切非常自然)

回答这个问题，我建议大家先来这样思考我们的项目，想象成一个表格，或者数据库表，也就是一个二维的矩阵，一个维度是不同的业务实体，在这里就是不同种类的campaign；另一个维度与它垂直，就像是auth, i18n, theming，routing等等，你每一行代码的作用，就都可以用一个向量来表示：我这一行，是处理DSA的i18n。那么两种思考问题的方式其实就是一个地方不一样：把哪个维度看成行，把哪个维度看成列。

传统的方法是根据业务来思考的。而这里推荐的另一个看问题的角度，是通过可扩展性，可维护性，（说人话：呼应标题，让前端开发变简单）的方面来思考。这和数据库很类似：列扩展很简单，一条SQL语句即可，行扩展相对麻烦。所以我们用middleware来描述我们的列:
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




