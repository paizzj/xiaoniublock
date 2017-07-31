>bool AppInitMain(boost::thread_group& threadGroup, CScheduler& scheduler)      
我们先看函数的两个参数，threadGroup, 和 scheduler. 从字面意思理解前者是
线程组(threadGroup), 后者是调度器(scheduler), 这个时候可以想象下线程组和
调度器协同工作的场景: 调度器去调度线程组中的线程协同工作。
是不是可以理解为比特币就是由线程组和调度器组成的？       
