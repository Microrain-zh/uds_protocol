# Autosar通信栈：基础问题知多少

积土而为山，积水而为海。——荀子

## QA1：CAN报文发送，有优先级吗？

**答**：有。以英飞凌tc3xx系列为例，MCMCAN模块有多个硬件发送缓冲区，也就是说同一时刻可以缓存多个待发送的报文，这些报文放入待发送的缓冲区以后，会置位发送Pending标志，等待发送，谁先发送呢？在回答这个问题之前，我们先说待发送的报文在硬件缓存区中的存储方式：Dedicated Tx Buffers 、Tx FIFO、 Tx Queue。

对于Dedicated Tx Buffers与 Tx Queue两种存储方式，根据lowest Message ID原则发送，即：发送的CanID数值越小，优先级越高。具体发送顺序示例如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzoAgzzT5e9T6WOIYgSMsLicRb5hPLWc6C63KwezqyGUPWsicQ0GG1xibaxPlyT8wF74ODhZRnz3VLYQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

对于Tx FIFO存储方式，报文的发送顺序由进入FIFO缓存区的先后顺序决定，即：先进先出，具体发送顺序如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzoAgzzT5e9T6WOIYgSMsLicHB28mCFAxriblnSfzBFuYoZvzdTU7PwSACF8EUJsPA5osBw9eKzzqTA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如果使用Dedicated Tx Buffers与 Tx Queue两种存储方式，优先级低的报文(比如：网络管理报文、诊断报文、标定报文等，优先级比应用报文低），是不是永远得不到发送了？不是，我们要清楚：发送的硬件缓存区不止一个，而是多个（比如：tc397有32个发送缓冲区），足以在某一时刻缓存多个待发送的报文。是否存在某一时刻（如下t1），发送报文的个数超过硬件缓存区个数？这种可能性是存在的，这也是为什么会有发送报文丢帧的原因。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzUWHKOibQVqT0GBibugKVCOia3BhGbjJ130ib0yjdHYzFtJBgDAiaX0iacG9gb2s8T84QyDn93wK3Q45LA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

为了避免发送丢帧或者周期性报文抖动问题，我们可以将**相同周期性报文**的发送做一个Offset处理，**避免周期性相同的报文以同一个基准时间计时**。比如：通信启动后，周期性报文Msg01(周期10ms)在t0时刻开始周期性发送；而周期性报文Msg02(周期10ms)在t1时刻开始周期性发送，这样即可避免两者在某一时刻的发送重叠，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzUWHKOibQVqT0GBibugKVCOiaF9oYKU4L6FC5F5ibM9UFvyX18eEoPmubvX1Hz2ryGuJWDqIEicJgodpA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## QA2：Tx FIFO发送方式可能引发的问题有什么？

**答：**对于CAN报文，使用FIFO方式发送，我能想到的场景：诊断中，发送诊断指令使用FIFO缓存，保证诊断指令请求的顺序。不知道大家在何种情况下使用过Tx FIFO，还请大家给普及。

这里思考到了一种工况，可能会因使用FIFO发送方式，导致报文的发送延迟，具体如下所示：

假设：CAN BUS上有两个节点：ECU1::CAN1和ECU2::CAN1，ECU1::CAN1使用Tx FIFO缓存待发送数据，ECU2::CAN1使用Dedicated Tx Buffers方式缓存待发送数据。在某一时刻，ECU1::CAN1的Buffer Index0待发送报文的CAN ID = 0x30，ECU2::CAN1待发送的6个报文的CAN ID ＜0x30，所以，每次总线仲裁，都会优先发送ECU2::CAN1缓存的报文，而ECU1::CAN1因为总线仲裁失败，导致ECU1::CAN1 FIFO中的高优先级报文无法及时发送出去，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzoAgzzT5e9T6WOIYgSMsLicOJJiaGZfNP2bibHiaiaGftZaQTv9vvXfPLo0JSFDYqluyhTOuU7D2yERibA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

所以，在使用Tx FIFO方式时，需要注意此工况。

## QA3：程序应先处理发送报文还是应该先处理接收报文？

**答**：TBD。为什么是未定义呢？在实际的项目开发中，先处理Rx报文还是先处理Tx报文确实没有明确规定，在项目不出问题之前，没有人会留意两者的处理顺序。这里我们讨论一下先处理Rx Msg，再处理Tx Msg的情况，实质就是讨论Task中，Com_MainFunctionRx()/Com_MainFunctionTx()的处理顺序。

假设：Com_MainFunctionRx()/Com_MainFunctionTx()均在5ms的Task中，且先处理Com_MainFunctionRx()，再处理Com_MainFunctionTx()，如果此节点收/发的报文数量不多，任务在规定的时间内处理完（t1时刻之前），不会引发问题，如下所示：



![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzUWHKOibQVqT0GBibugKVCOiaQlYqPE37MoL2HUJ1SnD2FicKedubgx3DBHiar21jNibOE2ibE8ZAwv0SwA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如果当前节点接收的报文数量较多，Com_MainFunctionRx()消耗了任务处理的大部分时间，导致Com_MainFunctionTx()处理被Delay，可能会导致本节点发送报文的周期抖动问题，比如：本来10ms的Tx Msg，由于延迟导致Tx Msg＞10ms + 容差值，假设容差值±10%(1ms)，比如：实际Tx Msg周期13ms，导致通信出问题。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzUWHKOibQVqT0GBibugKVCOiaibiaib2EGRKckNRSuqm5RFzaZrDBFYlVreWDonMuJaWtBquE9Dm364g0Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

对于一个节点，**接收报文**数量存在不确定性，比如：接收的报文类型如果有事件性应用报文、高周期应用报文（如：1ms周期性应用报文）等。那么Com_MainFunctionRx()的处理时间就会存在一定的不确定性。相对于接收报文，节点发送报文的数量相对比较确定，发送所消耗的时间也比较确定，所以，从处理顺序上来说，**先处理确定的发送再处理不确定的发送比较合理**，这样可以确保发送报文的时间。而对于接收，即使超一点时间，如果Task没有超过Deadline Time，对程序的运行也不会造成太大影响。

再者，对于CAN报文，发送报文还需要进行总线仲裁，仲裁失败也会存在一定的延时（参考QA2）。

**注意**：上述假设OS所使用的基准计数器是可信的，即：基础计数器准确，一般由GPT或者STM驱动。

## QA4:周期型报文Offset的作用是啥？

**答：**降低同一时刻，多个发送报文的Burst Send问题。这个问题属于QA1的延申。

一个节点，发送的报文类型可以有多种（QA1提到）。其中，节点外发的应用报文从几个到几十个不等。应用报文又分为事件型、周期型、混合型。以周期型应用报文为例，可能有5ms、10ms、20ms、50ms等周期。如果本节点外发的报文数量较大，在某一时刻会存在大量并发请求。比如：MsgA Cycle =5ms，MsgB Cycle = 10ms，MsgC Cycle = 10ms，如果MsgA的发送时刻为5ms、10ms、15ms、20ms、25ms、30ms.....MsgB、MsgC的发送时刻为10ms、20ms、30ms.....，在time　＝　10ms、time　＝　30ms......等时刻，MsgA 、MsgB、MsgC会同时请求发送，节点要发送的报文数量越多，某一时刻（如下图time = 10ms 时刻），请求发送的报文数量可能就越多。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vxWkjOHrD2yuu5hSeiaCUjmvnyG6lDzocogIWjYFKsibYyKOMbVDbS2k77gn2rgoErq37MH50C7DfwA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在某一时刻，发送报文数量过多会带来什么问题呢？这就是QA3中的问题，会使得某个发送报文发送延迟，使得其发送周期出现抖动，即：超过该周期报文的允许误差值，一般来说，≤100ms的周期性应用报文允许的偏差为3％，＞100ms的周期性应用报文允许的偏差为1％(看OEM要求)。既然应用报文会Burst Send，如何降低这种并发请求行为呢？**偏移相同周期报文的发送时机**，比如：MsgB　Offset 5 ms，MsgC Offset 10ms，这样两者的发送就不会重叠，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vxWkjOHrD2yuu5hSeiaCUjmvTtExtB1ibQo6dFsribQB6iaTlK0YXnjyrCeOt3ic5IvXamKtCTFdOQKjZw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

那偏移不同周期的应用报文有用吗？因为周期性报文发送取决于Com_MainFunctionTx()所在任务周期，Offset Value = n * Cycle（n为非负整数，Cycle指Com_MainFunctionTx()的任务周期），所以，Offset Value是Com_MainFunctionTx()任务周期的整数倍。这样，即使设置MsgA、MsgB的Offset Value不同，也不能避免某一时刻（如：time = 10ms时刻），两者的并发请求，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzoAgzzT5e9T6WOIYgSMsLicLsEiaQe8709BXQ3iamHn6nxTEG9HTyLttdWtpxRdd5UEQdKP3Mc1SpCw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



![图片](https://mmbiz.qpic.cn/mmbiz_png/quuOyCqONwdD16hI11wAQWxGp4ajJ4DMnbsHGb4ViandFryeibcQb1Idxb3MHmrh20988OSES3OU1wPTicQbAr94g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



参考资料

SIMPLE TITLE

Infineon-AURIX_TC3xx_Part2-UserManual-v01_00-EN.pdf