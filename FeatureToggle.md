# 功能触发器 - Feature Toggles

项目需要我们引入功能触发器，来控制一个新功能的开与关。团队成员有了不少讨论，对Feature Toogle的认识大相径庭，所以我想第一步还是要对它追本溯源，看一看这个东西原始用意是什么；第二步再结合自己项目的情况，来确定我们接下来的开发中如何使用它。于是翻来Martin Fowler在其个人网站上援引的[针对Feature Toggles文章](https://martinfowler.com/articles/feature-toggles.html)一探究竟，下面的内容除非注明，均是作者Pete Hodgson的原意，不包含我的评价和理解，当然由于我没有word by word的进行翻译，也可能会由于英文理解造成错误，所以强烈建议有能力的读者去读[原文](https://martinfowler.com/articles/feature-toggles.html)。

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

我们来做一个最简单最直观的Toggle:  
```javascript  
  function reticulateSplines(){
    var useNewAlgorithm = false;
    // useNewAlgorithm = true; // UNCOMMENT IF YOU ARE WORKING ON THE NEW SR ALGORITHM
  
    if( useNewAlgorithm ){
      return enhancedSplineReticulation();
    }else{
      return oldFashionedSplineReticulation();
    }
  }
  
  function oldFashionedSplineReticulation(){
    // current implementation lives here
  }
  
  function enhancedSplineReticulation(){
    // TODO: implement better SR algorithm
  }
```

当我们需要打开新功能时，我们就把第一行代码注释掉，同时把第二行的注释去掉，那么新功能就启用了。一个最简单的toggle就诞生了。

### 动态化该Toggle  
上面的toggle显然是粗糙的，很快我们的自动化测试人员开始抱怨起来，显然我们既要测试老SR算法（Jacky：因为生产环境还在用）也要测试新SR算法（Jacky：因为将来它们需要正确工作），在上面的toggle的基础上，在不改变代码的条件下，不可能测到两种算法。很快，我们写出下面的代码：  
```javascript
  function reticulateSplines(){
	if( featureIsEnabled("use-new-SR-algorithm") ){
	  return enhancedSplineReticulation();
	}else{
	  return oldFashionedSplineReticulation();
	}
  }
  function createToggleRouter(featureConfig){
    return {
      setFeature(featureName,isEnabled){
        featureConfig[featureName] = isEnabled;
      },
      featureIsEnabled(featureName){
        return featureConfig[featureName];
      }
    };
  }
```  
上面这段代码引入了一个“配置” - use-new-SR-algorithm, 它的值可能是通过其它手段由外部指定的。同时这里我们引入了一个“Toggle Router”，它的作用是获取或计算进而传递配置。有了上面这中新的toggle，我们的自动化测试脚本可以这么写,新老算法都会被测到：  
```javascript
describe( 'spline reticulation', function(){
  let toggleRouter;
  let simulationEngine;

  beforeEach(function(){
    toggleRouter = createToggleRouter();
    simulationEngine = createSimulationEngine({toggleRouter:toggleRouter});
  });

  it('works correctly with old algorithm', function(){
    // Given
    toggleRouter.setFeature("use-new-SR-algorithm",false);

    // When
    const result = simulationEngine.doSomethingWhichInvolvesSplineReticulation();

    // Then
    verifySplineReticulation(result);
  });

  it('works correctly with new algorithm', function(){
    // Given
    toggleRouter.setFeature("use-new-SR-algorithm",true);

    // When
    const result = simulationEngine.doSomethingWhichInvolvesSplineReticulation();

    // Then
    verifySplineReticulation(result);
  });
});
```
Jacky注：Toggle Router是一个重要概念，它会为系统的其它部分提供toggle的当前值，相当于一个single source of truth。那么它需要在合适的时候去获取这些toggle的值。由于系统内它是唯一的toggle提供者，这也为我们集中控制toggle提供了便利，例如根据环境背景（如那个用户）去决定一个toggle值，这就是可能了。  在上面的例子里面光看代码可能很难看出Toggle Router是怎么在系统里工作的，我想可能是需要很高的JS知识才能猜测出来。

### 准备发布  
随着时间的推移，我们的开发渐渐完成，我们不再简单满足于自动化测试用例都通过，我们已经准备好在线上去实验新算法的功效了。那么第一个问题就是我们以什么形式来改变feature toggle的值呢？我们可以在三个层次上组织toggle的修改：  
- Toggle的值被写进一个配置文件，不同的环境有不同的配置文件，在向这个环境部署代码的时候也会把对应的配置文件部署上去。而我们的Toggle Router会去读当前的配置文件，从而得到当前环境的toggle值。我们会在测试环境的配置文件中打开新SR算法的toggle，而在生产环境关闭。  
- 引入一个配置UI，管理员可以通过这个UI改变当前application的toggle值。一旦新SR算法被管理员打开，整个系统的用户都会用到它。    
- 我们改变Toggle Router的代码，让它更强大一点儿，能够分析当前user request以及当前环境，从而决定一个toggle的值。例如通过检测http header，检测cookie项等等。这最为灵活。

下面这张图描述了这三种不同方式：  
![toggle配置处](images/ft1.png)

经过讨论，我们项目组选择了第三种，基于user request的toggle。大家觉得这既保护了在线的老版本正常服务，也为我们项目组提供了在生产环境测试的可能。