&emsp;第四年了，推荐相关具体的工作和去年做的并无太大区别，主要还是集中在中日韩的日活和营收上，但这两年我一直有个主线任务，或者说是主线包袱，运行到现在再无事发生，我看我终于算是把这部分技术债和流程债补完了，总结一下。<br>
<br>
&emsp;在最开始给某些特定市场上线推荐系统的时候，我们和tt用了一样的策略，tt和抖音的架构和数据是分家的，我们也是，在完成主要框架迁移的过程里，有一些技术栈或者模型因为数据源尚未完成准备或者复杂度过高，我们是手动屏蔽了这一部分的patch，fork了一个ranker层和容器层，在最开始fork这个版本的时候我们组内就有一些争论，在权衡了agility和dev cost之后我们选择了fork,这也符合当时的需求，我们会有一部分合规需求，在整体框架和代码不可控的情况下，由团队按时间周期打包和统一检查新技术栈是最安全的，当时整个系统是新上线，任何合规问题都会被放大，这个策略确保了新的市场不会因为任何合规问题被下掉，而这很可能是不可逆的。<br>
<br>
&emsp;在fork一个版本出来之后，自然而然就需要有人打patch和追patch，这个任务当时我们组轮流来做，最开始的追patch主要也是把模型和树的版本更新，把一些新的策略加到我们负责的市场里来，这个过程可以总结为 收集patch -> 分析patch可用性 -> AB test ->上线。<br><br>
&emsp;大概这样过了半年之后事情就变得麻烦了起来，由于推荐系统整体步入正轨，大家更新迭代的速度越来越快，同时组与组之间的依赖变得更多，开始出现一些技术依赖于一些其他技术，并且技术默认配置到了全球，与此同时，容器层也在更新，容器层和ranker层的交互也变多了起来，这导致了我们负责的特定市场必须加快patch的频率才能确保整体系统的稳定性，随着时间的推进，打包变得越来越难以忍受，于是我们组讨论之后有了一个新的计划给我，换言之，我们要把tt和抖音重新合并。<br>
<br>
&emsp;合并这套系统里主要分为三部分，ranker/容器/计算资源合并，流程合并，监控的搭建。这是个数百人的团队，这些改动如果失败导致回滚，会非常影响数据科学家和研发的实验进度，我们只能一次做完。<br>
<br>
&emsp;Ranker层合并：这部分的合并是最简单的，因为过去半年多一直在做这件事，我们要做的就是做完一次AB test确保patch表现正常的时候，在尽可能短的时间里（3-5天）把其他部分合并。<br>
<br>
&emsp;容器层合并：这涉及先整个ranker代码的重构，但大部分工作量都被重构团队接手了，我们只需要配置我们特定市场代码逻辑的重构，并确保在新容器里能够正常工作。再把计算资源合并回全球市场。<br>
<br>
&emsp;计算资源合并：由于降本增效，团队没有额外的计算资源做迁移,整套系统需要在运行时做迁移，也没有备份。我大概花了一个月准备playbook和文档，最后花了五天分批把全球五个集群的计算资源迁移，主要的策略是 在低流量时保留少量计算资源，迁移特定市场计算资源到全球市场，迁移成功之后再将特定市场流量迁移过去，具体的动作过于危险，我也不想再来一次了，失败了就又是一次大的公关事件，我愿意以这张图来形容,虽然当时已经计划的很完备，但最后依然出了一些小状况，这也是一些经验教训。迁完计算资源还发现了一些额外的好处，全球各地的时差导致的计算峰值差异，合并资源让整条曲线变得更加平滑，降低了10%的CPU峰值占用。<br>
<br>
![image](https://github.com/user-attachments/assets/80ea56e5-7b45-411e-ae34-43cd10d41854)<br>
&emsp;流程合并主要分为三块，合规合并,在现有ranker的框架上多封了一层合规层，确保在 监控/上线/AB实验分析里都能够体现合规的动向，这涉及到从文章离线标注到使用再到Kafka消息的聚合，在持续集成里我也添加了相关的测试，能够在研发checkin代码之前做一些简单的合规测试，同时合规的大头主要是放在了最后ABtest的校验上，我在这块也用到了聚合的数据来做合规检查，最后的线上监控分为了两组策略：lookback检查过往24小时的合规状态，lookforawrd 检查当前系统的返回值合规状态<br>
<br>
&emsp;在这个基础上，我也迁移了我们整套监控逻辑，并且把一些其他组缺的监控补上了，到此为止迁移就做了一半了。我们需要在实际线上去测试迁移完之后的组件和框架能不能正常使用，在这个期间：确保其他数百人的研发不被我们影响，新技术的发布能够正常进行，系统整体不出现大的性能退化，这里我用了两套策略来实现这个办法，一个是shadow flight,一个是holdout flight, 路程过于漫长，此处按下不表。<br>

最后的最后，到了2024年年底，我发现我终于从这一系列事务里解放出来已经有三个月了，技术债总归是要还的，有时候，这也是一种合规和agility妥协的艺术。但在组里的强力支持下（工时/交流）这一系列事情终于有了尾声。这件事是在推荐生涯里一个插曲，抛开正常的ranking tech和data analysis以及各种新技术，这件事属于集体，我一开始也没有觉得能该做它到最后做完了，于是记录下这个插曲，偶尔也客串了一把后端工程师。

# 第三年了，把推荐做过的东西汇总
![Central Topic](https://github.com/SeanWeiSean/ProblemSummary/assets/17775798/52f6018d-f2bd-47a9-9303-c8c02c33122b)

# 两年中遇到的小问题汇总集合
![image](https://user-images.githubusercontent.com/17775798/173532961-3d9f5ae9-5e13-4bef-990b-eb9a58abe66b.png)
![image](https://user-images.githubusercontent.com/17775798/173532983-00195090-9371-48ff-a2a4-9cb248c1cebd.png)
![image](https://user-images.githubusercontent.com/17775798/173533034-a6bc5552-0637-4526-b5fa-2d66423b39a0.png)
