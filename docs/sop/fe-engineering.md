---
title: 对前端工程化的理解
date: 2023-03-14 09:23:13
tags:
categories: 工程化
description: 工程化是一种思想，而不是某种技术。其主要目的为了提高效率和降低成本，即提高开发过程中的开发效率，减少不必要的重复工作时间等
---

工程化是一种思想，而不是某种技术。其主要目的为了提高效率和降低成本，即提高开发过程中的开发效率，减少不必要的重复工作时间等

举个例子：要盖一栋大楼，假如我们不进行工程化的考量那就是一上来掂起瓦刀、砖块就开干，直到把大楼垒起来，这样做往往意味着中间会出现错误，要推倒重来或是盖好以后结构有问题但又不知道出现在哪谁的责任甚至会在某一天轰然倒塌，那我们如果用工程化的思想去做，就会先画图纸、确定结构、确定用料和预算以及工期，另外需要用到什么工种多少人等等，我们会先打地基再建框架再填充墙体这样最后建立起来的高楼才是稳固的合规的，什么地方出了问题我们也能找到源头和负责人。

那么前端工程化需要考虑哪些因素呢？

应该从模块化、组件化、规范化、自动化4个方面去思考

## 模块化

模块化就是把一个大的文件，拆分成多个相互依赖的小文件，按一个个模块来划分

## 组件化

页面上所有的东西都可以看成组件，页面是个大型组件，可以拆成若干个中型组件，然后中型组件还可以再拆，拆成若干个小型组件

- 组件化≠模块化。模块化只是在文件层面上，对代码和资源的拆分；组件化是在设计层面上，对于UI的拆分
- 目前市场上的组件化的框架，主要的有Vue，React，Angular2

## 规范化

在项目规划初期制定的好坏对于后期的开发有一定影响。包括的规范有

- 目录结构的制定
- 编码规范
- 前后端接口规范
- 文档规范
- 组件管理
- Git分支管理
- Commit描述规范
- 定期codeReview
- 视觉图标规范

## 自动化

也就是简单重复的工作交给机器来做，自动化也就是有很多自动化工具代替我们来完成，例如持续集成、自动化构建、自动化部署、自动化测试等等



## 前端工程化主要内容

以下是前端工程化的主要内容和目标：

1：代码管理：使用版本控制系统（如 Git）来管理代码，实现代码的版本管理、分支管理和协作开发。

2：构建工具：使用构建工具（如 webpack、Parcel、Gulp 等）来自动化构建过程，包括代码打包、资源优化、转译、压缩等，以提高开发效率和代码质量。

3：模块化开发：使用模块化的开发方式（如 ES6 模块、CommonJS、AMD 等）来组织代码，提高代码的可维护性和重用性，同时实现代码的按需加载和依赖管理。

4：自动化测试：采用自动化测试工具和框架（如 Jest、Mocha、Karma 等）来编写和运行单元测试、集成测试和端到端测试，确保代码的质量和稳定性。

5：代码规范和静态分析：使用代码规范工具（如 ESLint、Prettier）对代码进行格式化和静态分析，以保持代码风格的一致性，减少错误和提高代码质量。

6：性能优化：通过优化代码结构、减少网络请求、使用缓存、懒加载等手段来提升网页性能和用户体验。

7：文档和知识管理：建立文档和知识库，记录项目的架构、设计决策、接口文档等信息，方便团队成员之间的沟通和知识共享。

8：部署和持续集成：使用自动化部署工具和持续集成服务（如 Jenkins、Travis CI、GitHub Actions 等）来实现代码的自动部署和持续集成，提高开发和发布的效率。