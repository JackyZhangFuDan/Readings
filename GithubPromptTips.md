# Github Copilot 提示词撰写技巧
LLM在编程领域极大提高了开发人员的生产力，诸多公司已经主动拥抱这一趋势，为员工采购了编码辅助工具，而市场上最知名的自然是Github Copilot。笔者所在公司早在去年就开始普及Github Copilot，使用下来确实很震撼，编程工作很多时候变成了写提示词，然后一路按tab键 :-D. 本文总结了Github团队专家所分享的提示词撰写小技巧，笔者认为很有用。

## Github Copilot的工作模式  
Github Copilot的工作模式如下图所示，编辑器将相关提示词信息以及其它背景信息收集起来发给它的后端，经过后端的转发，请求OpenAI的模型进行代码提示

![bounded context](images/github-copilot.png)

## 有效提示词的核心因素  
1. 背景（Context）：更多的信息可以帮助Copilot更好地理解任务
2. 意图（Intent）：明确地描述你的特定意图，Copilot将产生更准确的结果
3. 清晰（Clarity）：模糊的描述不可能产生理想的结果
4. 具体（Specificity）：信息的详细程度、具体程度很重要

## 具体技巧  
1. 为变量起意义明确的名字
2. 函数、方法的签名要明确，参数名字有意义
3. 命名规则要保持一致性，例如一致用驼峰
4. 对于函数、方法期待的输入与输出描述清楚
5. 明确指出需要错误处理的场景，以及怎么处理
6. 可以描述控制流的具体过程，就相当于用语言把处理逻辑说一下
7. 最好首先（针对方面、模块）给出高层次的概述，让Copilot了解
8. 让你的提示词简介和有针对性
9. 结果不理想时，多迭代修改提示词，直到理想。这也有助于积累提示词撰写技巧
10. 保持和当前代码紧密相关的5、6个文件在编辑器的其它tab打开，Copilot会读取它们作为背景信息
11. 在提示词中包含例子。例如你希望输出是一个json，那么用例子描述一下这个json结构

## 总结
上述只是简单罗列了专家给的建议，并没有举例，不过每一条建议也不难理解。笔者觉得正确看待Copilot特别重要：你把他当作一个合作伙伴，你提问他回答，如果你问题好那么回答也会好。所以说到底还是问问题的艺术啊。