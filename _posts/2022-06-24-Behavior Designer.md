## Behavior Designer



##### 一 为什么用Behavior Designer

​	在Unity中，可以使用自带的Animator动画状态机来控制角色的动画，以及深层次的Blend Tree来制作角色的动画。

​	行为少的情况下还是可以勉强接受，但是一旦行为过多，就非常的错综复杂，想要理清楚非常困难，肉眼可见的抗拒去理解。

![.](/images/Behavior Designer1.png)

Behavior Designer 通过行为树的方式，理清楚了每一条逻辑，每一个分支，做什么行为，逻辑判断等情况。

这是游戏项目《KTV大亨》中收银员的行为树结构，每一个分支，每一个条件逻辑都非常的清晰。

![.](/images/Behavior Designer2.png)



##### 二 介绍相关组件

Composites类组件：非行为类型组件，用于引导流程走向

- Parallel：并行走向，即组件之下的所有分支全部并行
- Selector：组件之下从左至右进行逻辑判断，左侧不满足条件下会依次执行右侧的行为
- Sequence：组件之下从左至右依次执行流程



Decorators类组件：改变行为结果类型组件

- Inverter：反转，例如返回相反的bool值
- Repeater：重复执行，一个行为执行完毕后重复执行
- Return Failure：无论bool值是什么最终都返回false
- Return Success：无论bool值是什么最终都返回true



Actions组件：最终执行的行为组件，自定义编辑的逻辑也归为这一类型

Conditionals组件：与Actions组件类似，都是属于行为组件，但是更多的是用于条件判断



##### 三 实例介绍

这里以《KTV大亨》中收银员的部分逻辑作为演示

首先说下其行为，游戏开始后，女收银员检查自身体力是否满足，如果满足，就去工作，如果不满足就回休息室休息，直到体力补充满足后再继续工作。

这里的行为对应的是这一部分

![.](/images/Behavior Designer3.png)

![.](/images/Behavior Designer4.png)

行为讲解：

- 1 为为行为入口
- 2 为并行流程，因为无论该角色处于什么状态，碰倒了客人都要概率性的向客人问好，弹出对话框，具体逻辑在自定义脚本中编写。
- 3 从左至右的判断条件，即满足了5的分支，有体力可以工作，那么就不用执行6的分支去休息
- 4 为自定义脚本行为，遇见客人弹出对话框逻辑
- 5 从左至右依次执行流程
- 6 从左至右依次执行流程
- 7 Condition判断是否满足体力需求
- 8 如果满足了7 就会执行8 播放动画 接着进行14 之后的流程，这里不做讲解
- 9 如果7判断体力不足，就会执行6 然后执行9 寻找空出的休息位置
- 10 播放动画
- 11 向目标休息室走去
- 12 决定用什么姿态，站着还是坐着休息
- 13 休息的体力恢复逻辑



当游戏开始后，会进入1Entry入口，接着进入2Parallel并行 和3Selector选择判断，再就是5依次执行流程，进入5依次执行流程后会判断收银员的体力是否满足，如果返回的是True，就会继续执行8和14，收银员就会正常工作。如果返回的是False不满足，那么后面的8就不会再执行，直接中断返回False到3，由于5分支返回了false，就会执行另一个分支6的依次执行流程，开始寻找休息点，然后走过去开始休息，并原地休息，等待满足条件后重新该流程。

与此同时如果收银员遇见了客人也会执行并行的4 弹出对话框向客人问好。



##### 四 自定义脚本行为编辑

自定义脚本主要以Conditionals和Actions两类为主

Conditional为判断，需要返回成功或者失败，比如上面收银员的体力，在进入到此行为之前就要获取到相关数据来进行判断

```
    public SharedBool checkIsTrue;
    public override TaskStatus OnUpdate()
    {
        if (checkIsTrue.Value)
        {
            return TaskStatus.Success;
        }
        else
        {
            return TaskStatus.Failure;
        }
    }
```

而Actions行为 只需要在行为逻辑执行完成后返回Success即可

```
    public override TaskStatus OnUpdate()
    {
    	
    	 Debug.Log("执行逻辑");
    	
		 return TaskStatus.Success;
    }
```

