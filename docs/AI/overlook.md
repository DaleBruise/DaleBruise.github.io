# 1. 人工智能总览
本文讲介绍人工智能的含义、相关知识、分类及其需要的环境搭建。
## 1.1 什么叫人工智能？
智能，就是自主学习和解决问题的能力。人工智能，，就是机器模仿人类学习和解决问题的模拟。其本质而言，就是机器对人类的思维或者行为过程的模拟，让它能够像人一样行为和思考。人工智能能够通过输入，并对其进行处理之后，输出我们需要的信息，其有着一些特点：  
1、信息处理：人工智能能够对输入的信息进行处理，对不同的信息能够有着不同的处理方式，从而能够得到最终的我们想要的信息。  
2、自我学习：人工智能能够自动地对输入的信息生成其相应的处理方式。  
3、优化升级：人工智能能够对不断输入的新的信息和旧的信息相互结合，根据输入的信息进行不断地迭代和优化其处理方式。  
## 1.2 人工智能的发展阶段
我们所向往的人工智能是”强“人工智能，具有 1、真正的推理和解决复杂问题的能力，有类似人类的意识；2、其综合思考能力可以媲美或者超越人类。所以按照这个要求来看，我们目前的人工智能还是处于“弱”人工智能的阶段，1、是不具备真正的推理能力和复杂问题的解决能力的，是没有自己的意识；2、机器只能够基于事物的特征解决专一的能力，成为一类项目的专家或者资深助手，是一个强有力的工具。但是我目前认为机器是永远无法超越人类的意识，人工智能在发展的过程中，人们的意识也在不断地发展，人类的大脑只是需要一个能够记住上完本书本，并且能够随时随地应用的工具而已。  
## 1.3 人工智能的实现方法
人工智能主要通过符号学习和机器学习来实现。  
符号学习就基于固定的逻辑与规则的方法，来通过一定的符号语言来解决一个问题。例如使用do while循环解决线性单链表的插入。所以，符号学习是不具备自主优化的能力的，它不需要实时的大量数据来构建模型，都是预先按照数据设置的模型处理问题。  
机器学习比较复杂，是基于数据的学习方式。机器学习往往需要大量的实时数据来实现一个问题的解决。例如拟合线段、圆弧，预测房价的涨跌等等。机器学习也需要预先设置模型，但是在运行过程就可以通过实时输入数据来动态地解决问题。更加具有鲁棒性。  
人工智能通常是通过将这两种方法结合来实现，不是仅靠单一方法实现的。  
### 机器学习和深度学习的区别
深度学习被机器学习包含，深度学习具有机器学习的一切特征，并通过模仿人类的神经网络，具有更加复杂的问题解决能力，并且比较于一般的机器学习，更加具有鲁棒性，但是需要更多的确切的数据。  
## 1.4 开发环境的配置
编程语言：python  
开发辅助包：matbpoltlib，numpy，panda  
开发辅助工具：Jupyter  
# 2. 机器学习
本节主要介绍机器学习的部分基本概念。机器学习目前分为四类：监督学习、无监督学习、半监督学习和强化学习。这四类学习的区别在于，**监督学习**有一定的明确的分类方式，来对数据进行标识。而**无监督学习**没有具体的分类方法，而是通过数据本身具有的特征来进行分类，并对其进行标识。**半监督学习**则是这两种学习方法的结合。**强化学习**是通过对对行为进行奖惩，比如一个行为做对了就加分，反之则减分。  
## 2.1 监督学习  
监督学习有线性回归，逻辑回归，决策树，神经网络等等诸多应用方式。  
### 线性回归  

