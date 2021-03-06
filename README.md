# rawshare
![pic](https://github.com/firewolf-ljw/rawshare/blob/master/1.png?raw=true)

## 介绍 
用swift 1.2实现：不使用相关APP提供的SDK，分享信息（文本，图片，连接）到iOS的主流社交APP

本示例demo也是从github上的一个开源项目翻译过来的，网址：https://github.com/100apps/openshare
本人还根据自己的编程习惯做了一些修改。可能对swift掌握还不到位，若觉得有不妥的，还望指点。觉得可以的话，不妨施舍一颗星

源项目的作者还专门开了博客做推广：http://www.gfzj.us/series/openshare/
里面介绍了实现的思路，主要信息有：

 1.不同APP间是通过一个调起APP的URL和存放数据（如：图片）的系统粘贴板来实现通信的

 2.监控官方SDK的参数格式的方式是hook相关的方法，本项目中也提供了一段使用了方法绞合（Method Swizzling）来监控openURL方法的代码

## demo
本项目主要实现了将文本，图片，链接等信息分享到微信，QQ，微博的功能。
微信，QQ登录认证也都能工作，唯独微博的登录认证没跑通，因为没有去注册应用，不过登录认证的界面是调起来的了。
支付相关代码也都翻译完了，但由于没有相应的服务端，所以这两个都没有跑通。有兴趣，有时间的朋友可以帮我试试并修正


省去了SDK，大大减小了应用的大小，本demo的代码可以很方便的集成到其他应用里面去，步骤一下几步

1.在AppDelegate的应用加载完毕的回调方法里面，注册需要将信息分享到的应用接口，需要提供相应开放平台的appid等信息
	    
	    func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
        
        RSWeChat.register(ShareIdentity.WeChatAppId)//分享到微信
        RSQQ.register(ShareIdentity.QQAppId)//分享到QQ
        RSWeibo.register(ShareIdentity.WeiboAppKey)//分享到微博
        
        return true
    }

2.在触发分享信息事件的回调方法中调用相应的分享接口，将信息分享出去
	    
	    @IBAction func shareWeChat(sender: UIButton) {
        	if let share = ShareManager.getShare(domain: RSWeChat.domain) as? RSWeChat {
            	share.shareToWeChatSession(self.wcMsg(1),
                	success: { message in println("success")},
                	fail: { message, error in
                    	println("fail")
                    	println(error)
                	}
            	)
        	}
    	}

3.信息分享完成后，社交APP会回调我们的应用，只需在AppDelegate中的回调方法里做处理就可以了

	    func application(application: UIApplication, openURL url: NSURL, sourceApplication: String?, annotation: AnyObject?) -> Bool {
        
        if let share = ShareManager.getShare(scheme: url.scheme!) {
            return share.handleOpenURL(url)
        }
        
        return false
    }

4.注：修改Info.plist添加URLSchemes，社交APP会回调分享信息的应用时，系统才能匹配到我们的应用

	<key>CFBundleURLTypes</key>
	<array>
    	<dict>
        	<key>CFBundleURLName</key>
        	<string>OpenShare</string>
        	<key>CFBundleURLSchemes</key>
       		<array>
            	<!--       微信         -->
            	<string>wxd930ea5d5a258f4f</string>
            	<!--        QQ         -->
            	<string>tencent1103194207</string>
            	<string>tencent1103194207.content</string>
            	<string>QQ41C1685F</string>
           		<!--		微博			-->
            	<string>wb402180334</string>
        	</array>
    	</dict>
	</array>



## 参考链接
http://www.gfzj.us/series/openshare/ 

https://github.com/100apps/openshare 

http://nshipster.com/method-swizzling/ 

http://nshipster.cn/swift-objc-runtime/ 
