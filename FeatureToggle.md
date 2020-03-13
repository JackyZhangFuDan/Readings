# 功能触发器 - Feature Toggles

项目需要我们引入功能触发器，来控制一个新功能的开与关。团队成员有了不少讨论，对Feature Toogle的认识大相径庭，所以我想第一步还是要对它追本溯源，看一看这个东西原始用意是什么；第二步再结合自己项目的情况，来确定我们接下来的开发中如何使用它。于是翻来Martin Fowler在其个人网站上援引的[针对Feature Toggles文章](https://martinfowler.com/articles/feature-toggles.html)一探究竟，下面的内容除非注明，均是作者Pete Hodgson的原意，不包含我的评价和理解，当然可能会由于英文理解造成错误，所以强烈建议有能力的读者去读[原文](https://martinfowler.com/articles/feature-toggles.html)。

> Feature Toggles 是一种强大的手段，它允许我们在不更改代码的情况下改变系统行为。Feature Toggles有多种不同类型，我们有必要分清这些类型。同时Feature Toggles会引入额外的复杂性，我们有必要用聪明的手段去实现toggle管理toggle，即便是这样，尽量减少Toggle的数量也是我们必须重视的问题。

## 从一个例子看Toggle  
场景是我们有一个在开发的大项目“town planning simulation game” - 模拟小镇规划游戏，我们的团队负责一个部分。一天，我们接到一个任务：优化一下已经存在的“Spline Reticulation”算法。我们发现这并不是一件简单的事情，需要较多开发工作所以需要时间，与此同时，其它的团队还在工作，它们依赖于SR算法所以它要“在线”，我们的开发不能影响它们的工作。另外我们这个小组依照单分支的原则工作，我们不会有时间跨度很长的分支或者说我们所有人都在主分支（master branch，trunk）上工作。在这些要求和客观条件下我们该怎么开展工作呢？ 看来引入一个Feature Toggle是很好的选择。  

### 引入Toggle  
假设开始时SR算法长这个样子 (JS代码）  
```javascript  
	function reticulateSplines(){
    	// current implementation lives here
  	}
```

我们来做一个最简单最直观的Toggle