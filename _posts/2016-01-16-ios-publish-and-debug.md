---
layout: post
title: "iOS发布与调试"
category : iOS
tags : [iOS, game]
---


### 一、概念介绍

#### 1、开发者账号类型  

![publish01](http://edwinho.github.io/images/ios/publish01.png)

<!-- more -->

　　iOS 开发者计划主要为iOS设备进行App开发，比如iPhone和iPad等，iOS计划也是目前苹果整个开发者计划类型中人数最多的。账号类型分为个人（Individual）、公司（Company）、企业（Enterprise）、高校（University）四种类型，每年资费分别为$99、$99、$299、免费。根据账号类型的不同，申请的条件和所具有的权限也不同。  

　　个人计划只需要填写个人信息并通过苹果审核即可，公司计划需要出示相应的公司信息、营业执照等，企业账号需要出示的信息和公司账号类似，高校计划需要提供高校基本信息，在注册方式上苹果开发者官网有详细的流程。  

#### 2、代码签名（code signing ）  

　　对指定信息使用哈希算法，得到一个固定长度的信息摘要，然后再使用私钥（注意必须是私钥）对该摘要加密，就得到了数字签名。所谓的代码签名就是这个意思。  

　　代码签名的重要性：给应用程序签名是为了让用户相信这个APP 是已经通过苹果验证的开发者发布的，从而能够识别这个APP 是否是伪造的或者给别人修改过的。这样能够保证只有经过签名的应用程序才能保证他的来源是可信任的，并且代码是完整的、未经修改的。  


#### 3、证书和密匙  

　　①  证书类型(certificate)  

　　在开发 iOS 应用的时候，我们需要签名证书（开发证书）来验证，并允许我们在真机上对App 进行测试。另外，在发布App 到App store 的时候，我们也需要证书(发布证书)来做验证。所以在给你的app 签名之前，你应该准备好已经创建了调试证书或者发布证书。  

　　证书主要分为两类：Development 和 Production，Development 证书用来开发和调试应用程序，Production 主要用来发布应用程序（根据证书种类有不同作用），下面是证书的分类信息：  

![publish02](http://edwinho.github.io/images/ios/publish02.png)

* Development  
 * App Development:用来开发和真机调试应用程序  
 * Push Notification:用来调试Apple Push Notification  
* Production  
 * App Store  and Ad Hoc:用来发布提交App Store 的应用程序和发布测试应用程序  
 * Push  Notification(Production):用来在发布版本中使用 Apple Push Notifation  
 * Pass Type ID Certificate:用来在发布版中使用Passbook  
 * Website Push ID Certificate:用来在发布版中使用safri 的Push  
 * Watchkit Services Certificate:  
 * VoIP Services Creticate:  
 * Apple Pay Certificate:  


　　②  证书生成  

　　开发者在申请 iOS 开发证书时，需要通过 keychain 生成一个 CSR 文件（Certificate Signing Request），提交给苹果的  Apple Worldwide Developer Relations Certification Authority(WWDR)证书认证中心进行签名，最后从苹果官网下载并安装使用。这个过程中还会产生一个私钥，证书和私钥在keychain 中得位置如图：  

![publish03](http://edwinho.github.io/images/ios/publish03.png)

　　③  证书组成  

![publish04](http://edwinho.github.io/images/ios/publish04.png)

　　经过WWDR 数字签名后的数字证书分为两个部分：  

* 证书本身  
　　包含用户的公钥、用户个人信息、证书颁发机构信息、证书有效期等信息  
* 证书签名  
　　WWDR 将上述证本身的内容使用哈希算法得到一个固定长度的信息摘要，然后使用自己的私钥对该信息摘要加密生成数字签名  

![publish05](http://edwinho.github.io/images/ios/publish05.png)

　　④  证书使用  

　　iOS 系统原本就持有WWDR 的公钥，系统首先会对证书内容通过指定的哈希算法计算得到一个信息摘要；然后使用WWDR的公钥对证书中包含的数字签名解密，从而得到经过WWDR 的私钥加密过的信息摘要；最后对比两个信息摘要，如果内容相同就说明该证书可信。整个过程如图所示：  

![publish06](http://edwinho.github.io/images/ios/publish06.png)

　　⑤  公钥（public  key）  

　　公钥被包含在数字证书里，数字证书又被包含在描述文件(Provisioning File)中，描述文件在应用被安装的时候会被拷贝到iOS 设备中。  

　　iOS 安全系统通过证书就能够确定开发者身份，就能够通过从证书中获取到的公钥来验证开发者用该公钥对应的私钥签名后的代码、资源文件等有没有被更改破坏，最终确定应用能否合法的在iOS 设备上合法运行。  

　　⑥  私钥（private  key）  

　　每个证书（其实是公钥）都对应有一个私钥，  

　　私钥会被用来对代码、资源文件等签名。只有开发证书和描述文件是没办法正常调试的，因为没有私钥根本无法签名。  


#### 4 、Bundle ID 与App ID  

　　Bundle ID 能够精确的识别一个APP，这个和Android 的包名非常类似，相当于一个APP 的唯一标识。 Bundle ID 的使用涵盖从一个APP 的整个开发过程到可以安装到设备以及当APP 要发布面向消费者的时候提供给操作系统来使用。例如，当使用Game Center and In-App Purchase 这些服务必须使用bundle ID 来唯一标识你的APP。系统使用此字符串来确定哪些  

　　偏好设置来适合这个app，同样地，启动服务也用Bundle ID 来定位一个APP 可以打开一个特定的文件。Bundle ID 也可以用来验证APP 的签名。  

![publish07](http://edwinho.github.io/images/ios/publish07.png)

　　App ID 用于标识一个或者一组App，App ID 应该是和Xcode 中的Bundle ID 是一致的或者匹配的。App ID 主要有以下两种：   

* Explicit App ID：唯一的App ID，这种App ID 用于唯一标识一个应用程，例如com.ABC.demo1，标识Bundle ID 为com.ABC.demo1 的程序。   
* Wildcard App ID：通配符App ID，用于标识一应用程序。例如*可以表示所有应用程序，而com.ABC.*可以表示以com.ABC 开头的所有应用程序。    
　　每创建一个App ID，我们都可以设置该App ID 所使用的APP Services，也就是其所使用的额外服务。每种额外服务都有着不同的要求，例如，如果要使用Apple Push Notification Services，则必须是一个explicit App ID，以便能唯一标识一个应用程序。下面是目前所有可选的服务和相应的配置要求。  

![publish08](http://edwinho.github.io/images/ios/publish08.png)

#### 5、设备（Devices ）  

　　当需要将APP 在真实 iOS 设备上运行，如果你的设备没有越狱的话，你需要将设备通过 xcode 或者开发者管理中心来注册你的设备。Devices 中包含了该账户中所有可用于开发和测试的设备。 每个账户中的设备数量限制是100个。Disable 一台设备也不会增加名额。 每台设备使用UDID 来唯一标识。UDID（Unique Device Identifier）是区分物理设备的唯一标识。你的iPhone肯定有一个与众不同的UDID，通常UDID 会是一个40 位十六进制字符串。你可以通过xcode 或者itunes 来查看设备的UDID。  

![publish09](http://edwinho.github.io/images/ios/publish09.png)

#### 6、配置文件（Provisioning profies ）  

　　如果我们要打包或者在真机上运行一个应用程序，我们首先需要证书来进行签，用来标识这个应用程序是合法的、安全的、完整的等等；然后需要指明它的App ID，并且验证Bundle ID 是否与其一致；再次，如果是真机调试，需要确认这台设能否用来运行程序。而Provisioning Profile 就把这些信息全部打包在一起，方便我们在调试和发布程序打包时使用，这样我们只要在不同的情况下选择不同的profile文件就可以了。而且这个Provisioning Profile 文件会在打包时嵌入.ipa的包里。  

　　例如，如下图所示，一个用于Development 的Provisioning Profile 中包含了该Provisioning  Profile 对应的App ID，可使用的证书和设备。这意味着使用这个Provisioning Profile 打包程序必须拥有相应的证书，并且是将App ID 对应的程序运行到Devices 中包含的设备上去。  

![publish10](http://edwinho.github.io/images/ios/publish10.png)

　　如上所述，在一台设备上运行应用程序的过程如下：  

![publish11](http://edwinho.github.io/images/ios/publish11.png)

　　与证书一样，Provisioning Profile 也分为Development 和Distribution 两种，对应上面所提到的调试证书和发布证书：  

　　（注：前面提到不同账户类型所能创建的证书种类不同，显然Profile 文件的种类是和你所能创建的证书种类相关的）  

![publish12](http://edwinho.github.io/images/ios/publish12.png)

* In House：是企业账号发布的内部APP，使用In House 打包的APP 是无法发布到App Stroe 上面对外销售的，也不需要经过Apple 的评审。你可以把“In House”应用通过任何方式发布给你的企业员工、用户及其他你认可的任何人，没有注册设备的数量限制，尤其适合于企业应用的开发。   
* Ad Hoc：是用来发布测试用的或者是越狱版本，Ad Hoc 的包只能运行在该账户内已经注册的可用设备或者已经越狱的设备，最多 100 个注册设备的数量限制  

![publish13](http://edwinho.github.io/images/ios/publish13.png)

* App Store：是用来提交到App Store 上的，发布状态的Provisioning Profile 已经以签名的方式和 App 进行了绑定，有一点不同的是，发布状态的Provisioning Profile 不需要指定Device，因为它不知道将被哪些设备使用。  

![publish14](http://edwinho.github.io/images/ios/publish14.png)

### 二、开发/调试过程  

根据上面的介绍，可以知道进行Development 主要有以下几个步骤：

* 添加APP ID   
* 申请调试证书   
* 加入设备   
* 生成调试Provisioning Profile   
* 设置Xcode Code Sign Identifer，设置debug 选项为调试证书  

#### 1、 添加APP ID  

　　① 打开iOS Dev Center，选择Sign in，登陆（至少99 美元账号），登陆之后在网页右边找到iOS Developer Program,选择Certificates，Identifiers & Profiles。  

![publish15](http://edwinho.github.io/images/ios/publish15.png)

　　② 点击App IDs, 点击右上角的 “+”  

![publish16](http://edwinho.github.io/images/ios/publish16.png)

　　进入以下界面  

![publish17](http://edwinho.github.io/images/ios/publish17.png)

　　③ 选择Explict App ID  

![publish18](http://edwinho.github.io/images/ios/publish18.png)

　　④ 添加App Services  

　　应用程序提供的服务（App Services：选择你应用中将会使用的服务，在App ID 注册成功之后也可再次编辑你的选择） 在自己应用中所使用到的选项上打√，因为我选择的是精确型（Explicit)，所以Game Center，In-App Purchase， Push Notifications 都是可选的，勾选我们需要的APP Services，点击Continue。  

![publish19](http://edwinho.github.io/images/ios/publish19.png)

　　⑤ 确认信息，点击Submit 即可  

![publish20](http://edwinho.github.io/images/ios/publish20.png)

#### 2、 申请调试证书  

　　① 打开开发者管理首页选择Certificates，Identifiers&Profiles  

![publish21](http://edwinho.github.io/images/ios/publish21.png)

　　② 选择对应的证书类型  

![publish41](http://edwinho.github.io/images/ios/publish41.png)

　　点击+后，选择要创建的证书类型。  

　　③ 点击继续，会看到需要上传证书界面  

![publish22](http://edwinho.github.io/images/ios/publish22.png)

　　④  在mac 钥匙串的证书助手  

![publish23](http://edwinho.github.io/images/ios/publish23.png)

　　然后选择填写用户电子邮箱地址（随便填即可）  

　　请求必须选择存储到磁盘  

![publish24](http://edwinho.github.io/images/ios/publish24.png)

　　⑤ 点击继续后存储证书  

　　此时会创建一个名为CrtificateSigningRequest.certSigningRequst 的证书请求文件  

　　⑥ 在③的界面选择刚刚存储的证书上传；  

　　⑦下载证书，点击安装  

![publish25](http://edwinho.github.io/images/ios/publish25.png)

#### 3、 添加设备  

　　①  将iOS 设备连接到mac 上，打开itunes，获取设备的UDID  

![publish26](http://edwinho.github.io/images/ios/publish26.png)

　　②  回到Apple Developer，点击Devices，再点击右上角的”+ “号  

![publish27](http://edwinho.github.io/images/ios/publish27.png)

　　③  输入Name 和UDID，点击continue，最后点击Register  

![publish28](http://edwinho.github.io/images/ios/publish28.png)

#### 4 、 生成调试Provisioning Profile  

　　①  点击Development，点击右上角的 “+”  

![publish29](http://edwinho.github.io/images/ios/publish29.png)

　　②  选择iOS App Development 点击Continue  

![publish30](http://edwinho.github.io/images/ios/publish30.png)

　　③  选择我们一开始创建的APP ID，点击Continue  

![publish31](http://edwinho.github.io/images/ios/publish31.png)

　　④  选择刚才创建的调试证书，点击Continue  

![publish32](http://edwinho.github.io/images/ios/publish32.png)

　　⑤  选择刚才创建过的证书，点击Continue  

![publish33](http://edwinho.github.io/images/ios/publish33.png)

　　⑥  选择添加过的设备，点击Continue  

![publish34](http://edwinho.github.io/images/ios/publish34.png)

　　⑦  给这个描述文件起个名字，点击Generate  

![publish35](http://edwinho.github.io/images/ios/publish35.png)

　　⑧  成功提示，点击Download 下载到本地，双击安装  

![publish36](http://edwinho.github.io/images/ios/publish36.png)

#### 5、  设置Xcode Code Sign Identifer  

　　①  xcode 的左边项目导航区点击MGMClent.xcodeproj，点击Targets 的MGMClient，点击Build Settings，选择刚才下载下来的调试证书即可，这样就可以真机调试了  

![publish37](http://edwinho.github.io/images/ios/publish37.png)

### 三、发布流程  

Distribution 主要有以下几个步骤：   

* 申请发布证书   
* 生成发布Provisioning Profile  
* 设置Xcode Code Sign Identifer，设置release 选项为发布证书  
* 导出ipa  

　　前面1~4 步骤基本和调试过程一样，所以就不做太多描述，按着上面做就行，现在主要讲下导出ipa 的步骤：  

　　选择 xcode->Produce->archive,如果没有错误编译完成会见到以下界面，点击 Export 

![publish38](http://edwinho.github.io/images/ios/publish38.png)

　　后会出现：  

![publish39](http://edwinho.github.io/images/ios/publish39.png)

　　点击Next 后，选择创建的配置文件和对应的发布证书，即可导出ipa 文件，如下  

![publish40](http://edwinho.github.io/images/ios/publish40.png)