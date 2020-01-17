# Android 获取 SIM 卡信息

> 本文信息参考维基百科此条 [SIM card]( https://en.wikipedia.org/wiki/SIM_card)

SIM 卡上可存储的信息分为四类：

1. 固定存放的数据。在 SIM 卡被出售前由 SIM 卡中心写入，包括国际移动用户识别号（IMSI）、鉴权秘钥（KI）、鉴权和加密算法等等。
2. 暂时存放的和网络相关的数据。如位置区域识别码（LAI）、移动用户暂时识别码（TMSI）、禁止接入的公共电话网代码等。
3. 相关业务代码，如个人识别码（PIN）、解锁码（PUK）、计费率等。
4. 通讯录，用户随时存入的电话号码。不过现在很少有用户把号码都存储在 SIM 卡里了，因为 SIM 卡能存储的号码有限，而且不方便同步。

## 数据

SIM 卡上存储了特定于网络的数据，用于入网时的身份验证和标识。其中最重要的是 ICCID、IMSI、验证秘钥（Ki）、本地区域标识（LAI）和特定于操作人员的紧急号码。SIM 卡还存储了其他特定于电信运营商的数据，例如 SMSC（短消息服务中心）号，服务提供商名称（SPN），服务拨号号码（SDN），咨询收费参数和增值服务（VAS）应用。

SIM cards can come in various data capacities, from 8 KB to at least 256 KB. All can store a maximum of 250 contacts on the SIM, but while the 32 KB has room for 33 Mobile Network Codes (MNCs) or network identifiers, the 64 KB version has room for 80 MNCs.[citation needed] This is used by network operators to store data on preferred networks, mostly used when the SIM is not in its home network but is roaming. The network operator that issued the SIM card can use this to have a phone connect to a preferred network that is more economic for the provider instead of having to pay the network operator that the phone 'saw' first. This does not mean that a phone containing this SIM card can connect to a maximum of only 33 or 80 networks, but it means that the SIM card issuer can specify only up to that number of preferred networks. If a SIM is outside these preferred networks it uses the first or best available network.

### ICCID

每个 SIM 卡通过其集成电路卡标识符（ICCID）进行国际识别。ICCID 可以看做 SIM 的标识符。现在， ICCID 也可用来标识 eSIM 卡。





































































