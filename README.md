# PLL-design
for BLE under TSMC65G

转EETOP

1.STA分析，分析的Corner的选择
    在65nm的时候开始出现了温度反转的问题。在65nm以上的工艺的时候，应该说是在同一个电压下面，高温跑得慢，低温跑得快，而在65nm的时候就出现了，高温跑的快，低温跑得慢的问题。
  以TSMC65nm的工艺为例：
  就Library来说
有TT 1.1V, SS 0.99V, FF1.21V
对于RC来说有
cworst rcworst typrc rcbest cbest这几个模型，
那么分析Setup跟Hold应该在那个下面分析呢？
最直接的那么就是在最糟糕的那个点跑Setup，然后呢最快的那个点跑Hold,这个做法不能说她是错的，但是也可以说不是很科学。

从数学统计的角度来说， 同一个批次的芯片，在生产出来后，跑得快，跑得慢，跟跑得正常的Die，在数学上应该满足正态分布。就是说大部分的芯片都是跑在TT这个Corener的，跑在FF跟跑在ＳＳ的毕竟是少数。
那么说，在设计芯片的时候，Designeer就应该在SS，FF，TT定义出3套SDC。
那么在SS，TT，FF，那个Check Setup，那个是Hold呢。
如果你不知道哦啊，那么就用组合数学的方法来做。
Library的Corner+ 不同的RC值来做，来分析结果。
比如在SS Corner下的Cworst这个Corner呢，我同时分析cwost cbest，发现在125C的时候， cworst的SetupViolation最多，那么就应该把这个Corner定义成Check Setup的主要Corner。
同理其他的也是这么分析。
一般来说在Function mode下，会在SS下check　setup&Hold， TT check Setup , FF  Check Hold。
在Scan Mode下，一般就是在TT下面Check Setup & Hold。 应为跑Scan的时候，没有变态到把芯片加热后在来测试。如果有人这么干，可以把这个人炒了 。因为有点“二”

2. RC corelation
这个问题，可能大部分人都没有去考虑过。
问大家一个问题，EDA抽取到的RC参数跟真实的值对比，误差是多少？ 抽到的RC参数是不真实的值乐观，还是悲观呢？
大家比较常用的RC抽取的工具有Synopsys的StarRC跟 Cadence公司的QRC。
这两个工具抽取RC的参数的原理基本差不多，一个用2.5D来抽取，1个用3D的来抽取。似乎Cadence的QRC抽取到底参数更加准确。
古语有云“过犹不及”。
实际上QRC抽到的RC参数跟真实的值相比，QRC抽到的参数更加悲观，意思是它抽到的参数“过”了。、
而StarRC抽到的参数是不“及”。

那么这里我有一个问题问大家，现在我就用StarRC的抽到的RC参数用PrimeTime来分析Timing，应该如何解决？？？

大家都该知道，抽取RC参数，是一个比较费时间的过程。那么对一个比较大的Design，做RC参数的提取，RunTime你们有要求吗？
QRC抽取慢，
StarRC抽取参数快一点。
如果让你们选择这个过程的EDA工具，你们会怎么去选择？

3. DFM
   这个问题是一个比较大的问题。
   第一个Metel的密度，这个问题就很简单了。大家都知道。 如果Metel的密度没有达到一定程度，在做CMP的时候，会打磨得不够平整，或者说Densitiy底的地方，打磨得正好了，Density高度地方Metal就别磨没了。 所以每一次的Metal 密度有有一定的要求，在Routing完之后+Metal Fill 来把Metal密度增加上去，可以利用Calibre来Check每一层的Metal 密度。
第二,DFM Via 这个不说了，让工具去做吧。
第三, Fix CAA，通过spread wire来搞定。
弟四， 在GDS交付给Fouandary之后，其实还有文章可以做。对于Via Enclouse的Metal少的Via，可以在不违反DRC 规则上，可以增加Enclose的Metal，在Deck上做修改，这个是很有好处的，可以增加Yield。缺点是会影响到Timing。

OPC, 这个是跟光刻相关的东西，由于线宽越来越窄，Metal之间的距离约来越小， 本来两个没有连接关系的Metal就会链接上，或者有一根金属本来是连接上的，在被另外一个金属的影响下，就会断开。


TSMC定义了LPC(litho pattern correction,是不是这么叫有点忘记了) Hot-spot，从level -1 ~level -2.2, 在定义中level-1(necking & bridging) 的Hot-spot是致命的，一定要Fix掉，而level-2.1 ~level-2.3的可修可不修。 分析这种Hot-spot的过程叫CAA（Critical Area analysis)。 
OPC(光学近似矫正)，这个是考虑Lithto到影响下，Fix掉Litho引起的Routing的问题，Encounter在Routing中可以做这个东西，开启这个功能后，直接的结果是Runtime & Timing都变烂了。事实上如果两个比较靠近的Net，如果它们之间的距离够宽，Level-1的Hot-spot就会不存在了，也就是说，这个问题完全可以在Routing Rule中把这个问题解决掉。事实上我们在一个65nm一下工艺的一个项目中压根就没有考虑OPC到问题，估计OPC问题Foundary已经解决了，是在Tech File定义这种Routing的Rule吧。对于这点，我不是很sure，希望有高手来解决一下这个疑问！！！

