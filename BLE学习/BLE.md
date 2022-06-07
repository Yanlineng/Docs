#  广播相关参数

在 btstack/port/posix-h4 下make后，运行生成的例程的 gap_le_advertisment 可执行文件，通过 HCI_UART 控制蓝牙模块，查看空间中的蓝牙广播信息

低功耗蓝牙设备通过广播信道发现其他设备，一个设备进行广播，而另一个设备进行扫描。

广播相关的参数大致有以下几种：

- 1.Advertising interval
- 2.Advertising_Type
- 3.Own_Address_Type
- 4.Direct_Address_Type
- 5.Direct_Address
- 6.Advertising_Channel_Map
- 7.Advertising_Filter_Policy
- 8.Advertising Data
- 9.ScanReponse Data

## 1.1 Advertising interval

也就是广播间隔；设备每次广播时会在3个广播信道上发送相同的报文。这些报文被称为一个广播事件。除了定向报文以外，其他广播事件均可以选择“20ms~10240ms”的不等间隔（以0.625ms为单位）。两个相邻的广播事件之间的时间称为广播间隔。

这样会出现一个问题，严格按照设备周期性发送广播会由于设备间时钟会不同程度的漂移，因此导致两个设备在很长一段时间里同时广播，使信号收到干扰。为防止这一情况的发生，除定向广播之外的其他广播类型，发送时间均会被扰动。实现该扰动的方式为，在上一次广播事件后加入“0 ~ 10ms ”的随机延时。**这意味着，即使两个设备广播间隔相同，并在相同信道及时间点上发送造成了冲突，但它们发送下一个广播事件时也会有很大可能不再冲突。**

简单来说就是使用随机时延，尽量保证不发生广播冲突，或者尽可能减少广播冲突的影响。

广播包时间：

![](.\image\22.JPG)

两个相邻的广播事件的之间的时间间隔（T_Adv Event）为：

​																		T_Adv Event = advInterval + advDelay

adv Delay为随机时延（0 ~ 10ms）。

当然，实际设置过程中没有广播间隔参数，而是设置**Advertising_Interval_Min**（最小广播间隔）和**Advertising_Interval_Max**（最大广播间隔）这两个参数来调整广播间隔

如果广告事件类型是可扫描的无向事件类型（scannable undirected event）或不可连接的无向事件类型（non-connectable undirected event），则间隔不得小于100ms。

## 1.2 Advertising_Type

也就是**广播类型**；广播的类型一般分为四种：

![](.\image\20160416101646718.jpg)

1. 0x00 **可连接的非定向广播**（Connectable Undirected Event Type），应用最广泛，包括广播数据和扫描响应数据，表示当前设备可以接受其他任何设备的连接请求
2. 0x01 **可连接的定向广播**（Connectable Directed Event Type），定向广播类型是为了尽可能快地建立连接。报文中包含两个地址：广播者的地址 + 发起者的地址。发起者收到发给自己的定向广播报文后，可以立即发送连接请求作为回应。定向广播类型有特殊的时序要求。完整的广播事件必须每**3.75ms**重复一次。当使用定向广播时，设备不能被主动扫描。此外，定向广播报文的净荷中也不能带有其他附加数据。该净荷只能包含两个必须的地址。
3. 0x02 **可扫描的非定向广播**（Scannable Undirected Event Type）。这种广播**不能用于发起连接**，但允许其他设备扫描该广播设备。这意味着该设备可以被发现，既可以发送广播数据，也可以响应扫描发送扫描回应数据，但**不能建立连接**。这是一种适用于广播数据的广播形式，动态数据可以包含于广播数据之中，而静态数据可以包含于扫描响应数据之中。
4. 0x03 **不可连接的非定向广播**（Non-connectable Undirected Event Type）。仅仅发送广播数据。

注意：所谓的定向和非定向针对的是广播的**对象**，如果是针对特定的对象进行广播（在广播包PDU中会包含目标对象的MAC）就是定向广播，反之就是非定向。可连接和不可连接是指是否接受连接请求，如果是不可连接的广播类型，它将不回应连接请求。可扫描广播类型是指**回应扫描请求**。

不同的广播类型对扫描请求和连接请求的不同结果如下图：

![](.\image\20160416101646718.jpg)

## 1.3 Own_Address_Type

即自身地址类型；

![](.\image\20160416101841517.jpg)

1. 0x00 Public Device Address：**公有设备地址**是设备所特有的并且是不可改变的。类似网络设备的MAC地址，它的长度为48位。由两部分组成：

![](.\image\3.jpg)

2. 0x01 Random Device Address：**随机设备地址**（私有设备地址），它也是48位。

![](.\image\4.jpg)

## 1.4 Direct_Address_Type

即定向地址类型，和自身地址类型Own_Address_Type一样

## 1.5 Direct_Address

即定向地址

![](.\image\20160416102351137.jpg)

## 1.6  Advertising_Channel_Map

广播信道

![](.\image\20160416102515228.jpg)

一个广播事件中，一个广播包会在每个信道上进行传输。

## 1.7 Advertising_Filter_Policy

广播过滤策略，对发来请求包的设备采用的过滤策略。

![](.\image\20160416102553013.jpg)

1. 接受任何设备的扫描请求或连接请求。（Value：0x00）

2. 仅仅接受白名单中的特定设备的扫描请求，但是接受任何设备的连接请求。（Value：0x01）
3. 接受任何设备的扫描请求，但仅仅接受白名单中的特定设备的连接请求。（Value：0x02）

4. 仅仅接受白名单中的特定设备的扫描请求和连接请求。（Value：0x03）

5. 保留。

## 1.8 Advertising And Scan_Response_Data

广播数据和扫描回应数据，在蓝牙4.0中它们的长度都不能超过**31**个字节（0 ~ 31）可是在5.0中可以达到255个字节，数据的格式必须满足下图的要求，可以包含多个AD数据段，但是每个AD数据段必须由“Length : Data”组成，其中Length占用1个octet，Data部分占用Length个字节，所以一个AD段的长度为：Length+1。

![](.\image\20160416103047452.jpg)

1 octet = 8 bit

# BLE

# 1 Link Layer 

## 1.1 states

设备的 Link Layer 有五种状态：

- Standby State : 待机状态，等待事务的到来
- Advertising State ： 广播状态，作为广播者广播发出数据包
- Scanning State ： 扫描状态，扫描监视广播中的数据包
- Initiating State ： 发起态，向其他设备发起连接的状态
- Connection State ： 连接态，已经与其他设备链接的状态，此状态中有2种角色： Master Role 和 Slave Role ， 如果是从发起态转移过来的，则为Master；从广播态则是Slave。

5种状态的状态机：

<img src=".\image\2.JPG" style="zoom:67%;" />

## 1.2 DEVICE ADDRESS

BLE中使用设备地址来标识蓝牙设备，设备地址长度为48 bits。 并且一个设备至少使用一种类型的地址，当然也可以同时包括2种类型。

### 1.2.1 Public Device Address

**公有设备地址**是设备所特有的并且是不可改变的。类似网络设备的MAC地址，它的长度为48位。由两部分组成：

![](.\image\3.jpg)

### 1.2.2 **Random Device Address**

**随机设备地址**（私有设备地址），它也是48位。

![](.\image\4.jpg)

这里面还分了2种，静态设备地址和私有设备地址，不多描述

## 1.3  AIR INTERFACE PACKETS

BLE 在空中进行数据传送，在 Spec 中称之为 **Air Interface packets**，俗称**空口包**。格式如下：

![](.\image\5.JPG)

- Preamble ----------------- 空口包的前导，PHY 层含义，具体与AA（Access Address）的LSB有关
- Access Address -------- 接入地址，用来标示接收者ID或者空中包身份(与 前文中的Address有区别)
- PDU ------------------------ protocol data unit 协议数据单元（广播报文的具体数据）
- CRC ------------------------- PDU 的 24 bits CRC 计算值，用于校验数据正确性   循环冗余检验码

单位为 octet = 8 bits

### 1.3.1 Preamble

前导码为0 1交替的形式，至于具体的是0101还是1010由Access Address中的LSB决定

### 1.3.2 Access Address

接入地址，与之前的设备地址无关。

在广播包中，AA变成了**固定值0x8E89BED6**，所以基本是依据PDU中的数据信息进行事务操作。

而在链路层连接中，AA则是由发起态的设备随机生成的32 bits的地址，用于连接接入，此时需要满足一些特殊的条件。这是区别任意两个设备的链路层连接的标志。

### 1.3.3  PDU

即为包的数据单元，有两个状态：在广播包中的PDU应该称为广播信道PDU，在数据传输包中，应该被称为数据通道PDU

## 1.4  ADVERTISING CHANNEL PDU

广播信道PDU中含有一个16-bit 的头部，以及剩下部分的有效载荷。

![](.\image\6.JPG)

### 1.4.1  Header

![](.\image\7.JPG)

头部的内容入上图：

- PDU Type: 表示ADV Type , 广播包的类型
- RFU : 预留
- TxAdd : 0 - 代表ADV是 public 类型的 Address ，否则为 1 - 代表 random 类型的 Address
- RxAdd : 0 - 代表期望对端地址类型为 public ，否则为1 - 代表期望对端地址类型为 random
- Length : 代表后面的 Payload 长度， 以 octet 为单位，也就是 6 bit ，则最大长度为32字节

**注意：**蓝牙5.0与这里有些不同，首先是在第一个RFU中取出了一位，变成了**ChSel** （跳频(Hopping)相关，本机支持则为1）；还有一处就是最后的RFU全部分给了Length字段（变成8 bits），所以PayLoad长度可以更大了，变成了**255字节**

### 1.4.2 Payload

<img src=".\image\8.JPG" style="zoom:80%;" />

上面的事件是出于广播态的设备发出，由扫描态或发起态的设备接收。

1. ADV_IND （Connectable Undirected Event Type）可连接的非定向网络广播事件；处于 Scanning 状态的设备收到本设备发出的 ADV_IND 后，对端发起 SCAN_REQ，并且本设备回复 SCAN_RSP。


![](.\image\9.jpg)

```
AdvA：本机地址 48 bits //由header中的TxAdd字段决定类型
AdvData：携带的数据 0 - 31 字节
```

2. ADV_DIRECT_IND （Connectable Directed Event Type）可连接的定向广播事件

   直接定向发送给对端，是可连接的并且带指向性的（不可扫描），携带本机地址以及对端地址，不可以携带数据。

![](.\image\10.jpg)

```
AdvA：本机地址 48bits   //由header中的TxAdd字段决定类型
InitA：对端地址 48bits  //由header中的RxAdd字段决定类型
```

3. ADV_NONCONN_IND（Non-connectable Undirected Event Type）不可连接的非定向广播事件；只能进行广播，不能连接，可以响应和处理广播事件。

![](.\image\11.jpg)

```
AdvA：本机地址 48bits   //由header中的TxAdd字段决定类型
AdvData：携带的数据 0 - 31 字节
```

4. ADV_SCAN_IND  （Scannable Undirected Event Type）可扫描的非定向广播事件；这种类型的 ADV PDU 是只能扫描的不定向的，不连接的 ADV

![](.\image\12.jpg)

```
AdvA：本机地址 48bits   //由header中的TxAdd字段决定类型
AdvData：携带的数据 0 - 31 字节
```

**以下类型称为扫描PDU：**

也是属于advertising physical channel，主要是为了与ADV交互的

5. SCAN_REQ （Scan Request Event Type） 扫描请求事件；由扫描态的设备发出，广播态的设备接收，不能携带数据

![](.\image\13.JPG)

```
ScanA：扫描端地址(本机) 48bits   //由header中的TxAdd字段决定类型
AdvA：广播端地址(对端) 48bits  //由header中的RxAdd字段决定类型
由于是扫描端的请求，所以容易理解本机地址是扫描端地址
```

6. SCAN_RSP  （Scan Response Event Type）： Advertising 回复 Scanner 的请求事件； 方向与上者相反

![](.\image\14.JPG)

```
AdvA：广播端地址(本机) 48bits  //由header中的TxAdd字段决定类型
ScanRspDate：扫描请求回应包的数据 0 ~ 31 字节
```

**以下类型称为发起PDU：**

7. CONNECT_REQ （Connect Request Event Type）: 连接请求事件，用于发起态的设备发出连接请求，由广播态设备接收

<img src=".\image\15.JPG" style="zoom:80%;" />

```
InitA：发起端地址(本机) 48bits  //由header中的TxAdd字段决定类型
AdvA：广播端地址(对端) 48bits   //由header中的RxAdd字段决定类型
LLDate: 数据部分 22 字节
AA： Access Address 接入地址  32 bits
CRCInit： 规定的CRC算法初始值，依据此进行连接双方的CRC检验  24bits
WinSize：传输窗口大小   8bits //transmitWindowSize = WinSize * 1.25 ms 
WinOffset：传输窗口偏移  16bits  // transmitWindowOffset = WinOffset * 1.25 ms
Interval：间隔时间  16bits   //connInterval = Interval * 1.25 ms
Latency：延迟时间 16bits   //connSlaveLatency = Latency
Timeout：超时时间 16bits   //connSupervisionTimeout = Timeout * 10 ms
ChM：数据通道使能位图 30bits  //每一位代表一个数据传输通道是否使用
Hop：数据信道选择算法中使用的跳点增量 5bits
SCA：最坏情况的睡眠时钟准确性 3bits
```

## 1.5   **DATA CHANNEL PDU** 

数据通道传输PDU中包含一个16bits的header，还包括可变长度的有效载荷部分，以及消息完整性检查的MIC字段

<img src=".\image\16.JPG" style="zoom:80%;" />

### 1.5.1  Header

5部分字段组成：

<img src=".\image\17.JPG" style="zoom:80%;" />

<img src=".\image\18.JPG" style="zoom:80%;" />

```
LLID: 2bits //决定了后面有效载荷的格式，有三种类型的数据格式
NESN: 1bit //下一个预期序号
SN: 1bit  //当前序号
MD: 1bit   //More Data 是否有更多数据
```

NESN 和 MD 可能都和数据包的拆分有关，据此可以还原完整的数据。

### 1.5.2   LL DATA PDU

LL DATA PDU是用来发送**L2CAP数据**的数据信道PDU，在Header的LLID字段中可能表示为01b或者10b。

当LLID为01b时，此时Length字段应该为00000000b，表示这是一个空的PDU，一般Master的Link Layer可以向从机的Link Layer发送空的PDU，以允许从机使用任何数据通道PDU进行响应，包括一个空的PDU。

而LLID为10b时，Length字段就不能为00000000b了，不能是一个空的PDU。

### 1.5.3   LL Control PDU

LL Control PDU是一种控制Link Layer层连接的数据信道PDU

![](.\image\19.JPG)

```
Opcode: 8bits //操作码
CtrData: 0 ~ 26 字节   //控制数据
```

LL  Control PDU的长度字段不得设置为00000000b。所有的LL控制PDU都有一个固定的长度，这取决于操作码。

操作码类型如下：

<img src=".\image\20.JPG" style="zoom: 67%;" />

<img src="C:\Users\yll\Desktop\BLE学习\image\21.JPG" style="zoom:67%;" />

## 1.6   NON-CONNECTED STATES

# 2 GATT

可以发现基本上GATT里面的信息条目都是使用的ATT的Attribute格式：

- Attribute Type: 由UUID表示
- Attribute Handle: 属性句柄
- Attribute Value: 属性值
- Attribute Permission: 权限相关：Access Permissions 决定Client是否可读、写属性值；Authentication Permissions 决定是否需要一个认证的物理链路(Authenticated Physical Link)；Authorization Permssions  决定Client在访问属性值前是否需要授权

所有的声明、定义都是单独的**条目**，不同的条目要按照不同的顺序进行存放。

## 2.1 Service declaration

<img src=".\image\24.JPG" style="zoom:80%;" />

## 2.2 Include definition

<img src=".\image\25.JPG" style="zoom:80%;" />

## 2.3 Characteristic Definition

A characteristic definition shall contain a characteristic declaration, a Characteristic Value declaration and may contain characteristic descriptor  declarations.

### 2.3.1 Characteristic declaration

<img src=".\image\23.JPG" style="zoom: 67%;" />

#### 2.3.1.1 Characteristic Properties

决定了可以以何种方式使用Characteristic Value，怎样访问Characteristic Description

<img src=".\image\26.JPG" style="zoom: 80%;" />

#### 2.3.1.2 Characteristic Value Attribute Handle

包含Characteristic Value 的 Attribute 的 Handle

#### 2.3.1.3  Characteristic UUID

是一个16bits 或 128bits 的UUID，用于区分Characteristic Value的type

### 2.3.2 Characteristic Value declaration

包含了Characteristic的Value，Characteristic Value 就是一个Attribute

![](.\image\27.JPG)

### 2.3.3 Characteristic Descriptor declaration

包含有关Characteristic Value的相关信息。由characteristic descriptor UUID区分。

#### 2.3.3.1 Characteristic Extended Properties

<img src=".\image\28.JPG" style="zoom: 80%;" />

<img src=".\image\29.JPG" style="zoom:80%;" />

#### 2.3.3.2 Characteristic User Description

<img src=".\image\30.JPG" style="zoom:80%;" />

#### 2.3.3.3 Client Characteristic Configuration*

<img src=".\image\31.JPG" style="zoom:80%;" />

还是在一个Attribute中，此条目出现在Characteristic Definition中，且在Characteristic Value之后。

需要注意的是Attribute Value中的**Characteristic Configuration Bits**，这里包含了配置信息，在下面一张图中显示，默认是0x0000。

每一个Client端都应该是配置自己的Characteristic Configuration，都有自己的实例，意味着，在Server端即使是对同一个Characteristic进行描述，每个Client都对应着自己的描述，不同的Client应该有独立存在的描述。

#### 2.3.3.4 Server Characteristic Configuration



## 2.4 Features

###  2.4.1  Server Configuration

用来配置ATT，只包含交换MTU子过程

#### 2.4.1.1 Exchange MTU

<img src=".\image\32.JPG" style="zoom:67%;" />

交换Client和Server双方的Rx MTU（接收）最大值，然后双方默认设置MTU为其中较小的一个值。ATT配置完成。

### 2.4.2  PRIMARY SERVICE DISCOVERY

主服务发现

#### 2.4.2.1 Discover All Primary Services

使用ATT中的 *Read By Group Type Request* 请求，

要发现全部主要服务，则使用ATT中的Read By Group Type Request，其中Attribute Type 应该设置为**《Primary Service》的UUID**，Handle范围设置为**0x0001到0xFFFF**。具体看ATT中的包格式。

<img src=".\image\33.JPG" style="zoom:67%;" />

#### 2.4.2.2 Discover Primary Service by Service UUID

已知UUID，通过ATT的*Find By Type Value Request*去发现服务，其中Attribute Type 设置为UUID for 《Primary Service》，Attribute Value设置为服务的UUID，Start Handle ~ End Handle设置为0x0001 - 0xFFFF（查找服务下的所有特征句柄）。

<img src=".\image\34.JPG" style="zoom:67%;" />

### 2.4.3  RELATIONSHIP DISCOVERY

依赖关系发现

#### 2.4.3.1 Find Included Services

查找包含服务，ATT中 *Read By Type Request* ，其中Attribute Type 设置为UUID for 《Include》

<img src=".\image\35.JPG" style="zoom:67%;" />

### 2.4.4 CHARACTERISTIC   DISCOVERY

#### 2.4.4.1 Discover All Characteristics of a Service

查找服务下所有Characteristic，ATT *Read By Type Request* ， 其中Attribute Type 设置为UUID for 《Characteristic》

<img src=".\image\36.JPG" style="zoom:67%;" />

#### 2.4.4.2 Discover Characteristics by UUID

ATT *Read By Type Request* ，其中Attribute Type 设置为UUID for 《Characteristic》， *Starting Handle* 和 *Ending Handle* 设为Handle的范围

<img src=".\image\37.JPG" style="zoom:67%;" />

### 2.4.5   CHARACTERISTIC   DESCRIPTOR    DISCOVERY

Discover All Characteristic Descriptors

<img src=".\image\38.JPG" style="zoom:67%;" />

### 2.4.6  CHARACTERISTIC VALUE READ

#### 2.4.6.1 Read Characteristic Value

*Read Request* ，  *Attribute Handle*参数是  *Characteristic Value Handle*

<img src=".\image\39.JPG" style="zoom:67%;" />

#### 2.4.6.2 Read Using Characteristic UUID

 *Read By Type Request* ，  Attribute Type 是 known characteristic UUID , *Starting Handle* 和 *Ending Handle* 设为Handle的范围

<img src=".\image\40.JPG" style="zoom:67%;" />

#### 2.4.6.3 Read Long Characteristic Values

读一个较长的值时，Client是知道 *Characteristic Value Handle*  以及 它的长度，此时它的长度大于一个单一 Response 包所能够携带的数据长度，此时需要使用ATT 中的  *Read Blob Request* ；

<img src=".\image\41.JPG" style="zoom:67%;" />

此时，Request中的Attribute Handle 是  *Characteristic Value Handle*  ，还需要设置当前请求的 Value Offset ，Value Offset是与请求的Characteristic Value相对位置有关的，第一个包中offset应该为 0x00，在途中可看到，连续的几个Request只有offset不同，直到Client 读到全部的Characteristic Value。

#### 2.4.6.4  Read Multiple Characteristic Values

前提是 Client 直到所有需要读的 *Characteristic Value Handle* ，使用 ATT 中的*Read Multiple Requests* ，且参数为需要请求的*Characteristic Value Handles* 的一个集合。返回包也会返回一个对应的集合。

此时需要注意，一个返回包最多能够返回 (ATT_MTU – 1) octets的数据。

<img src=".\image\42.JPG" style="zoom:67%;" />

### 2.4.7   CHARACTERISTIC VALUE WRITE

#### 2.4.7.1 Write Without Response

<img src=".\image\43.JPG" style="zoom:67%;" />

写 CHARACTERISTIC VALUE 时使用 Write Command； 包含 Charateristic Value Handle 和 需要写new *Characteristic Value*的目标值

此包不要求回应

#### 2.4.7.2 Signed Write Without Response

<img src=".\image\44.JPG" style="zoom:67%;" />

#### 2.4.7.3 Write Characteristic Value

已知  *Characteristic Value Handle* ， 使用 ATT 中的  *Write Request* ，包含 Charateristic Value Handle 和 需要写new *Characteristic Value*的目标值

写成功时需要返回一个  *Write Response* 

<img src=".\image\45.JPG" style="zoom:67%;" />

#### 2.4.7.4 Write Long Characteristic Values

写长值， 同时使用2个 Request 包 *Prepare Write Request* and *Execute Write Request* ，还需注意*Value Offset* 与 *Part Attribute Value* 搭配写值的一部分

<img src=".\image\46.JPG" style="zoom:67%;" />

此过程存在Error包，如果客户端使用的身份验证不足、授权不足、加密密钥大小不足，或者不允许对特征值进行写操作，服务器应通过响应准备写请求发送错误响应。

#### 2.4.7.5 Reliable Writes

<img src=".\image\47.JPG" style="zoom:67%;" />

所有的Client 的写请求第一阶段都会以预备指令暂存在Server端，等待Server端通知确认以及可以修改进行每一个请求（以返回包来通知），然后由Client发起**执行**指令，Server则会按照之前预备指令的**顺序依次**进行写入。

### 2.4.8  CHARACTERISTIC VALUE NOTIFICATION

关于Charateristic Value 的主动通知。

#### 2.4.8.1 Notifications

使用ATT 中的l *Handle Value Notification* 

<img src=".\image\48.JPG" style="zoom:80%;" />

### 2.4.9  CHARACTERISTIC VALUE INDICATIONS

关于Charateristic Value 的指示命令

#### 2.4.9.1  Indications

<img src=".\image\49.JPG" style="zoom:80%;" />

### 2.4.10 CHARACTERISTIC DESCRIPTORS

#### 2.4.10.1 Read Characteristic Descriptors

<img src=".\image\50.JPG" style="zoom:80%;" />

#### 2.4.10.2 Read Long Characteristic Descriptors

<img src=".\image\51.JPG" style="zoom:80%;" />

#### 2.4.10.3 Write Characteristic Descriptors

<img src=".\image\52.JPG" style="zoom:80%;" />

#### 2.4.10.4 Write Long Characteristic Descriptors

<img src=".\image\53.JPG" style="zoom:80%;" />

# 3  SM

The Security Manager 定义了 **配对**  和  **密钥分发** 的方法

<img src=".\image\54.JPG" style="zoom:80%;" />

Pairing 表现为构建起加密连接的密钥，分为三个阶段：

- Phase 1: Pairing Feature Exchange  （配对特性交换）
- Phase 2 (LE legacy pairing): Short Term Key (STK) Generation   （LE 传统配对  使用短期密钥STK）
- Phase 2 (LE Secure Connections): Long Term Key (LTK) Generation  （LE安全连接，使用长期密钥LTK）
- Phase 3: Transport Specific Key Distribution  （传输特定密钥，分发）

<img src=".\image\55.JPG" style="zoom:80%;" />

 ## 3.1 Pairing Feature Exchange

配对的过程总是以Pairing Request和Pairing Response的协议交互开始，通过这两个命令，配对的发起者（Initiator，总是Master）和配对的回应者（Responder，总是Slave）可以交换足够的信息，以**决定在阶段2**使用哪种配对方法、哪种鉴权方式、等等

### 3.1.1 配对方法

前面提到的2种：LE legacy pairing和LE Secure Connections

当Master和Slave都支持LE Secure Connections（新方法）的时候，则使用LE Secure Connections。否则，使用LE legacy pairing。

### 3.1.2 鉴权方式(Authentication)

所谓的鉴权（Authentication），就是要保证执行某一操作的双方（或者多方，这里就是配对的双方）的身份的合法性

(1) OOB(out of band)方式，带外方式，在配对过程之外，又配对的双方额外交互一些信息，依据这些额外的信息进行后续的配对操作。

(2) 让人参与进来

例如配对码，由人来对比比较，确定连接的双方具有同一个6个数字的配对码。在蓝牙协议里面称作MITM（man-in-the-middle）authentication。

- Passkey Entry，通过输入配对码的方式健全，由人输入
- Numeric Comparison , 两个设备自行生成6个数据，由人来比较确认是否在两个设备上都一样
- Just Work ，由两个设备自行协商，不需要人的参与

(3) 不需要使用鉴权

```
1）如果双方都支持OOB鉴权，则选择该方式（优先级最高）。
2）否则，如果双方都支持MITM鉴权，则根据双方的IO Capabilities（并结合具体的配对方法），选择合适的鉴权方式（具体可参考BT SPEC[3] “BLUETOOTH SPECIFICATION Version 4.2 [Vol 3, Part H]”“2.3.5.1 Selecting Key Generation Method”中的介绍）。
3）否则，使用Just work的方式（不再鉴权）。
```

LE Secure Connections中包含 OOB、Passkey Entry、Numeric Comparison 三种方式，而LE legacy pairing中只有OOB、Passkey Entry两种

### 3.1.3  IO Capabilities

Security Manager抽象出来了三种MITM类型的鉴权方法，这三种方法是根据两个设备的IO能力，在“Pairing Feature Exchange”阶段自动选择的。

<img src=".\image\56.JPG" style="zoom:80%;" />

<img src=".\image\57.JPG" style="zoom:80%;" />

IO的能力可以归纳为如下的六种：

- NoInputNoOutput
- DisplayOnly
- NoInputNoOutput1
- DisplayYesNo
- KeyboardOnly
- KeyboardDisplay

<img src=".\image\58.JPG" style="zoom:80%;" />

### 3.1.4 配对密钥生成方法选择

1. LE legacy pairing

<img src=".\image\59.JPG" style="zoom:67%;" />

2. LE Secure Connection Pairing

<img src=".\image\60.JPG" style="zoom:67%;" />

3. 依据 IO Capabilities

<img src=".\image\61.JPG" style="zoom: 80%;" />

<img src=".\image\62.JPG"  />



