# Android 中的 IPv6 

### 什么是IPv6

IPv6 的全称是**Internet Protocol version 6**。Internet Protocol 译为“互联网协议”，

所以 IPv6 就是**互联网协议第6版**。

它对比于 IPv4 所带来的是地址池的扩容，这是它比较重要的一个优势。

IPv4 的地址数量是2的32次方，而 IPv6 的地址数量是2的128次方，显然有明显的增大.。

IPv6 的地址长度是128位（bit），将这128位的地址按每16位划分为一个段，将每个段转换成十六进制数字，并用冒号隔开。

例如：2000:0000:0000:0000:0001:2345:6789:abcd

### IPv6 的生态发展

Apple 在2016年5月4日宣布，所有提交至 AppStore 的应用必须支持 IPv6 协议。而且，在 [Google IPv6 使用统计](https://www.google.com/intl/en/ipv6/statistics.html)中可以看出，当前 IPv6 的采用率已经接近 30%。说明 IPV6 正在高速扩张中，已经逐渐演变成未来互联网的一种发展趋势。

2017年11月，中共中央办公厅、国务院办公厅印发了《推进互联网协议第六版（IPv6）
规模部署行动计划》（以下简称《行动计划》），明确提出了未来五到十年我国基于 IPv6
的下一代互联网发展的总体目标。目前来说，国家正在大力推进 IPv6 的普及。

苹果系统在 iOS 12.1 版本后，安卓系统在 Android 8.0 版本后，已全面支持 IPv4/IPv6 双栈协议，其代理厂商也提供了相应的支持，包括华为，小米，oppo，一加等各大厂商等。

但是根据最新的国内 IPv6 研究报告表明，目前国内的 APP IPv6 可用度偏低。目前支持 IPv6 的网站及应用，大部分是首页可达，更深层次的链接还未支持 IPv6 访问，后续需加大 IPv6 改造深度。

而且相关使用资料较少，可能会遇到比较多坑。


### IPv6 的分配方式

IPv6 支持的地址分配方式：

1、 无状态地址自动配置

无状态地址自动配置是指主机通过监听路由通告获得全局地址前缀（64位），然后在后边缀上自己的接口地址得到全局IP地址。这主要是因为 IPv6 有海量的IP地址资源，用户可以自行配置一个全局 IP。

2、 有状态地址自动配置

有状态地址自动配置是由 IPv4 下的 DHCP 转化而来，IPv6 继承并改进了这种服务，即 DHCPv6 协议，它向 IPv6 主机提供有状态的地址配置或无状态的配置设置。

### IPv6的分配规则

知道 IPv6 地址分配方式之后，我们可以进一步的了解它的分配规则，首先是它的地址组成

IPv6地址 = 前缀 + 接口标识

- 前缀：相当于 IPv4 地址中的网络ID

- 接口标识：相当于 IPv4 地址中的主机ID

默认用于标识子网的 IPv6 地址前缀为64位

下面是 IPv6 的分配方案：

| 前缀长度 | 适用场景                                                     |
| :------- | :----------------------------------------------------------- |
| 32       | RIR/NIR（区域/国家互联网注册机构）分配给有 ASN 的运营商、互联网公司、大型企业。是地址最小分配单元（再小就不给了）。 |
| 40       | 运营商向有多个（256个以内）站点和数据中心的大型企业分配的前缀 |
| 44       | 运营商向有多个（16个以内）站点和数据中心的中型企业分配的前缀 |
| 48       | 运营商向中小客户分配的常见前缀长度。或大中企业内一个站点的前缀 |
| 56       | 宽带运营商给家庭用户和小微企业分配的最小前缀长度（最大子网大小） |
| 64       | 末端设备子网，/64 是很多协议硬性要求的（IPv6 无广播风暴风险） |
| 127      | 路由器点对点链路，此处不是为了节约地址而是防止一种资源耗尽型攻击 |

由于 IPv6 的接口地址部分，即后 64 位，所能容纳的地址数量远超过现有任何设备的硬件转发表项，可以近似看做无限的地址空间，完全不需要考虑节约地址的事情。

按照 IPv6 的分配规则，目前中国的运营商有两种前缀，一个是56位，一个是64位。

电信大都是56，联通移动大都是64的。（数据待考查）

不管是56还是64，你获得的剩余地址量都是用不完的，哪怕是给你家里每一粒灰尘都分配上公网IP。

竟然每一粒灰尘都有IP，那是不是可以说我们就拥有**固定的IP**了，其实不然。

在 IPv4 公网地址很充裕的时代，家用电脑都很少会有固定的公网IP，主要原因是运营商给家用电脑开启宽带时，采用的输入用户名和密码的 PPPoE 拨号的方式（就是给我们一个用户名密码，输入后拨号上网）。

而 PPPoE 拨号的方式分配地址的原则，就是运营商的宽带接入设备上会按照一定算法随机给你分配一个公网IP，所以你每次拨号，再下线，就会发现电脑获取的IP地址总是变化的。对于分配 IPv4 私网地址也一样，只要我们拨号上线再关机下线重新拨号，总会发现我们的IP地址是变化的，这是 PPPoE 的基本原理。

所以在IPv6时代，默认情况下使用 PPPoE 拨号的方式，肯定能够保证每个PC机拥有公网的 IPv6 地址，但是无法保证每个PC机都有固定 IPv6 地址。除非你的PC机采用 PPPoE 加上 DHCPv6 的方式，和运营商签约，才有可能有固定IP地址。同时一台路由器连接多个电脑，每个电脑都会有多个 IPv6 地址。

### IPv6的通信方式

Android的通信方式：

为了区分 IPv4 和 IPv6 地址，Java 提供了两个类：Inet4Address 和 Inet6Address，它们都是InetAddress 类的子类。

 这两个类分别按着 IPv4 和 IPv6 的规则实现了 InetAddress 类中的 public 方法。它们所不同的是Inet6Address 类比 Inet4Address 类多了一个方法：isIPv4CompatibleAddress，这个方法用来判断一个 IPv6 地址是否和 IPv4 地址兼容。和 IPv4 兼容的 IPv6 地址除了最后四个字节有值名，其他的字节都是0，如0：0：0：0：0：0.192.168.18.10、：：ABCD：FAFA都是和 IPv4 兼容的 IPv6 地址。

 当使用 InetAddress 类的四个静态方法创建 InetAddress 对象后，可以通过 getAddress()  返回的byte数组来判断这个IP地址是 IPv4 还是 IPv6 地址（byte数组长度为4就是 IPv4 地址，byte数组长度为16就是 IPv6 地址），也可以将 instanceof 来确定 InetAddress 对象是它的哪个子类的实例。

可以使用下面的代码判断是否是 IPv6 的地址

```java
InetAddress address = InetAddress.getByName(host);
System.out.println("IP: " + address.getHostAddress());
switch (address.getAddress().length)
 {
   case 4:
    System.out.println("根据byte数组长度判断这个IP地址是IPv4地址!");
    break;
   case 16:
    System.out.println("根据byte数组长度判断这个IP地址是IPv6地址!");
    break;
  }
 if (address instanceof Inet4Address)
   System.out.println("使用instanceof判断这个IP地址是IPv4地址!");
 else if (address instanceof Inet6Address)
   System.out.println("使用instanceof判断这个IP地址是IPv6地址!");

```

在 Java 层面上来说，只要获得相应的IP地址，就可以通过 Socket 进行网络通信

伪代码如下:

```java
 //获得Ipv6的地址
 String server = getIPv6Address();
 //客户端Socket
 Socket socket = new Socket(server, 8080);
 //服务端Socket
 ServerSocket serverSocket = new ServerSocket(8080);
 Socket client = serverSocket.accept();
```

我们要做的兼容可能是移动 App 经常使用 RESTful URL 与 服务端进行通信，而服务端 App 可能使用 URL 与 LDAP 服务数据库或其他 RESTful 服务通信。由于 IPv6 地址以 ":" 为分隔符，而 host 和端口也是同样的分隔符，所以在 URLs 中使用 IPv6 需要使用方括号加以区分： https://[2111:500:4:13::128\]:443/ 。

通过网络库 OkHttp 源码可以发现它也是通过匹配字符串" [ ",'' ] ''以及字节长度来判断是否是IPv6的地址，如果是的话，进行必要的规范和压缩。

```kotlin
/**
 * If this is an IP address, this returns the IP address in canonical form.
 *
 * Otherwise this performs IDN ToASCII encoding and canonicalize the result to lowercase. For
 * example this converts `☃.net` to `xn--n3h.net`, and `WwW.GoOgLe.cOm` to `www.google.com`.
 * `null` will be returned if the host cannot be ToASCII encoded or if the result contains
 * unsupported ASCII characters.
 */
fun String.toCanonicalHost(): String? {
  val host: String = this

  // If the input contains a :, it’s an IPv6 address.
  if (":" in host) {
    // If the input is encased in square braces "[...]", drop 'em.
    val inetAddress = (if (host.startsWith("[") && host.endsWith("]")) {
      decodeIpv6(host, 1, host.length - 1)
    } else {
      decodeIpv6(host, 0, host.length)
    }) ?: return null
    val address = inetAddress.address
    if (address.size == 16) return inet6AddressToAscii(address)
    if (address.size == 4) return inetAddress.hostAddress // An IPv4-mapped IPv6 address.
    throw AssertionError("Invalid IPv6 address: '$host'")
  }

  try {
    val result = IDN.toASCII(host).toLowerCase(Locale.US)
    if (result.isEmpty()) return null

    // Confirm that the IDN ToASCII result doesn't contain any illegal characters.
    return if (result.containsInvalidHostnameAsciiCodes()) {
      null
    } else {
      result // TODO: implement all label limits.
    }
  } catch (_: IllegalArgumentException) {
    return null
  }
}
```

通过以上分析，可以了解到 IPv6 的通信方式，先获取到相应的IP地址，然后进行规范化，达到符合 IPv6 的标准，再将这个 URL 进行 Socket 通信。

而服务端，由于 IPv6 地址有许多不同的格式，不推荐使用正则式来识别 IPv6。可以尝试使用一些开源库来做这部分工作。另外，应检查一下存放 IP 地址的数据库字段是否已经扩展到 128 位，确保你的应用部署到了支持双协议栈的操作系统中，确保你使用的库也支持 IPv6，否则可能出现不必要的错误。

### IPv6的数据意义

IPv6 地址数量海量，可以给每个上网设备分配一个全球唯一的IP地址，这样的IP地址就可以有效溯源。IPv6 地址就会和电话号码一样，从号码前几位就知道用户是从哪里注册的，就显示出你的身份信息，因为每一个地址都是真正独一无二的，相当于从技术上为每个人分配了一个“网络身份证”。每个人或者每个设备都有这样的一张身份证，通过这张身份证能迅速找到它所在的位置，以及它的周围环境信息和网络特征，实现精准定位。IPv6 还对源地址有一套验证体系，可以更好满足金融级应用所要求的身份验证和抵御网络攻击的能力，在安全性方面IPv6有质的提升。IPv6 不仅IP地址长，IP头也长，IPv6 不再采用IPv4地址固定的20字节报文头，而是可以为IPv6增加一些可选头，这些可选头IPv6可带可不带，完全取决于应用需要。



##### 文中如有错误，欢迎指正~

