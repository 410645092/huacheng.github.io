# Unity Job System



#### 一  Job System与多线程

官方解释：你可以多线程处理不同类型的任务，只要任务不需要在主线程外访问游戏对象即可。Job System允许我们轻松编写多线程代码，从而实现高性能游戏体验。它不仅能改善帧率，而且在做移动开发时，它还能显著改善移动设备的电池寿命。通过该功能，我们能够编写和Unity引擎功能共享工作线程的代码。

我的理解：Job System就是多线程管理系统，我们通过Job System来编写与Unity组件进行交互的相关多线程代码，并且可以提供更好的性能，用以解决多个物体与对象的运算逻辑造成的性能问题。



##### 一句话比喻Job System就是，皇帝把政务分发给六部各自同步执行，不必一条一条自己处理了。



##### 什么是多线程？

在单线程计算系统中，一次执行一条指令，并且一次产生一个结果。加载和完成程序的时间取决于 CPU 需要完成的工作量。多线程是一种编程方式，利用了 CPU 在多个核心上同时处理多个线程的能力。这种情况下不是逐个执行任务或指令，而是同时运行任务或指令。默认情况下，在程序的开头会运行一个线程。这就是“主线程”。主线程会创建新线程来处理任务。这些新线程并行运行，通常在完成后将其结果与主线程同步。如果有一些长时间运行的任务，这种多线程方法很有效。但是，游戏开发代码通常包含许多一次执行的小指令。如果为每个指令创建一个线程，最终可能会有许多线程，每个线程的生命周期都很短。这种情况下可能会挑战 CPU 和操作系统处理能力的极限。



##### NativeContainer

在使用JobSystem时，需要将结果储存在一种共享内存中，这是一种托管值类型，未本机内存提供了一个相对安全的封装器。



##### 创建Job

Job必须最为一个结构体

其成员变量必须是 NativeContainer或Blittable类型之一，一般我们使用NativeContainer

结构中需要创建一个Execute方法，在其中实现需要的运算。

```
public struct MyJob : IJob
{
    public float a;
    public float b;
    public NativeArray<float> result;

    public void Execute()
    {
        result[0] = a + b;
    }
}
```



##### 作业的调度运行

首先需要实例化该Job

给Job中的数据赋值

调用方法进行运算

Schedule作为调用运算的方法只能在主线程调用

```
// 创建单个浮点数的本机数组以存储结果。此示例等待作业完成，仅用于演示目的
NativeArray<float> result = new NativeArray<float>(1, Allocator.TempJob);

// 设置作业数据
MyJob jobData = new MyJob();
jobData.a = 10;
jobData.b = 10;
jobData.result = result;

// 调度作业
JobHandle handle = jobData.Schedule();

// 等待作业完成
handle.Complete();

// NativeArray 的所有副本都指向同一内存，您可以在"您的"NativeArray 副本中访问结果
float aPlusB = result[0];

// 释放由结果数组分配的内存
result.Dispose();
```



#### 二 Unity中使用

using Unity.Jobs;  

using Unity.Collections; 

需要引入相关的命名空间



接下为了方便采用一个比较简单的遍历累加数值

遍历10000次0-10000的累加

```
 int count = 10000
 for (int i = 0; i < count; i++)
 {
               
      int num = 0;
      for (int j = 0; j < count; j++)
      {
         int index = j;
         num += index;

      }
 }
```
![](https://github.com/410645092/huacheng.github.io/blob/master/images/NoJob.png?raw=true)


可以看到整个计算花费255+毫秒。



接下来使用Job System的方式来编写代码

将需要做的计算写入一个结构体中，IJob作为一个接口，当Job执行时会调用Execute来执行每一个分配的Job任务



```
//创建Job
public struct CalculatePlusJob : IJob
{
    int count ;
    public void Execute()
    {
        count = 10000;
        int num = 0;
        for (int i = 0; i < count; i++)
        {
            int index = i;
            num += index;

        }
    }
}
```

```
    //让Job System执行的方法  调度Job
    JobHandle JobHandles() {

        CalculatePlusJob job = new CalculatePlusJob();
       return  job.Schedule();
    }
```

```
	最终的运行函数
	void Calculate()
    {
        float starttime = Time.realtimeSinceStartup;
        int count = 10000;
        if (UseJob)
        {  
        	//首先创建多个调度Job的控制列表
            NativeArray<JobHandle> handlist = new NativeArray<JobHandle>(count, Allocator.Temp);
            for (int i = 0; i < count; i++)
            {
            	//将每一个即将运行的Job加入到控制列表中，同时开始进行调度计算
                JobHandle handle = JobHandles();
                handlist[i] = handle;

            }
            //等到所有的Job全部完成
            JobHandle.CompleteAll(handlist);
            //当所有的Job完成后释放
            handlist.Dispose();
            float timer = (Time.realtimeSinceStartup - starttime) * 1000f ;
            UseJobSystem.text = string.Format("使用Job System花费{0}ms", timer);
        }
        else
        {
         
            for (int i = 0; i < count; i++)
            {
               
                int num = 0;
                for (int j = 0; j < count; j++)
                {
                    int index = j;
                    num += index;

                }
            }
            float timer = (Time.realtimeSinceStartup - starttime) * 1000f;
            NoJobSystem.text = string.Format("未使用Job System花费{0}ms", timer);
        }

       
    }
```

使用Job System后的效果 只需要45+毫秒

![](https://github.com/410645092/huacheng.github.io/blob/master/images/UseJob.png?raw=true)
