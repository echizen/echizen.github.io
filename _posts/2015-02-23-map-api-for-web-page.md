---
layout: post
title: "网页调取APP"
description: "map api for web page，网页调取高德地图、百度地图，高德地图、百度地图URL api使用，网页调取原生地图API"
category: tech
tags: [pro, js]
---
{% include JB/setup %}



其实高德地图、百度地图的JS API写的很清楚，但是对于如何调用，没玩过的会觉得有点迷茫，最近项目要调用地图，我也是看了半天，才理解为何网页能有如此的能耐调用手机内安装的APP。
#基础篇

我们能在浏览器内跳转网页是http、https协议，能打开本地文件是file协议，调用APP用的是scheme协议或者intent协议，其实都是发送请求，获得响应，只是网页和APP是使用这两个协议通信的。scheme协议不仅仅用于ios端，也适用于安卓。但intent只适用于安卓。

##1、scheme协议
在 iOS 里，程序之间都是相互隔离，目前并没有一个有效的方式来做程序间通信，幸好 iOS 程序可以很方便的注册自己的 URL Scheme，这样就可以通过打开特定 URL 的方式来传递参数给另外一个程序。原生的APP和网页通过这种协议可以互相通信，safari或者APP的webView都可以识别该协议（一些浏览器如微信内置浏览器和QQ浏览器不支持，据后台技术哥哥介绍这是腾讯有意屏蔽，默认的webview控件是支持的）.

如果是自己的APP想被网页调用，需要自己在app中注册scheme，这不是我研究的范畴。。。

我们一般是调用别人的APP，会使用scheme协议就行,其实特简单，**通过链接就可以**。

###常用APP的scheme
QQ   mqq://   
微信  weixin://   
淘宝  taobao://   
大众点评  dianping:// dianping://search   
新浪微博  sinaweibo://   
支付宝 alipay://   
豆瓣fm    doubanradio://    
美团  imeituan://     
1号店 wccbyihaodian://   
有道词典yddictproapp://   
知乎  zhihu://  
优酷  youku://

见知乎上的整理：[http://www.zhihu.com/question/19907735](http://www.zhihu.com/question/19907735)

##2、intent协议
“在一个Android应用中，主要是由四种组件组成的，这四种组件可参考“Android应用的构成”。而这四种组件是独立的，它们之间可以互相调用，协调工作，最终组成一个真正的Android应用。在这些组件之间的通讯中，主要是由Intent协助完成的。Intent负责对应用中一次操作的动作、动作涉及数据、附加数据进行描述，Android则根据此Intent的描述，负责找到对应的组件，将Intent传递给调用的组件，并完成组件的调用。因此，Intent在这里起着一个媒体中介的作用，专门提供组件互相调用的相关信息，实现调用者与被调用者之间的解耦。”所以说intent是协议是不准确的，intent实际上是一种在不同组件之间传递的请求信息。

#示例：网页调取百度地图、高德地图

官方文档：

1、高德地图：[http://lbs.amap.com/api/uri-api/ios-uri-explain/](http://lbs.amap.com/api/uri-api/ios-uri-explain/)

2、百度地图：[http://developer.baidu.com/map/index.php?title=uri/api/ios](http://developer.baidu.com/map/index.php?title=uri/api/ios)

例如调用常规线路规划导航：

##scheme调用

高德地图：
`<a href="iosamap://path?sourceApplication=applicationName&backScheme=applicationScheme&sid=BGVIS1&slat=39.92848272&slon=116.39560823&sname=A&did=BGVIS2&dlat=39.98848272&dlon=116.47560823&dname=B&dev=0&m=0&t=0">公交导航</a>`

百度地图：
`<a href="baidumap://map/direction?origin=34.264642646862,108.95108518068&destination=40.007623,116.360582&mode=driving&src=yourCompanyName|yourAppName">驾车导航</a>`

##intent调用

**高德地图：**
`<a href="androidamap://route?sourceApplication=softname&slat=36.2&slon=116.1&sname=abc&dlat=36.3&dlon=116.2&dname=def&dev=0&m=0&t=1&showType=1">线路规划</a>`

**百度地图：**
`<a href="intent://map/direction?origin=latlng:34.264642646862,108.95108518068|name:我家&destination=大雁塔&mode=driving&region=西安&src=yourCompanyName|yourAppName">线路规划</a>`

注意：有些安卓机型上面2种方式都不适用。。。

#习惯性贴代码：
<script type="text/javascript" src="js/map.js"></script>
    
        <script type="text/javascript">
            var mapName = [amap,baidumap];
            var iosScheme = [];
            var androidIntent = [];
            iosScheme[0] = 'iosamap://path?sourceApplication=huiyouxing&backScheme=applicationScheme&sid=BGVIS1&slat=&slon=&sname=&did=BGVIS2&dlat='+initPosition.lat+'&dlon='+initPosition.lng+'&dname='+positionName+'&dev=0&m=0&t=0';

            iosScheme[1] = 'baidumap://map/direction?origin='+myLocation.lat+','+myLocation.lng+'&destination='+initPosition.lat+','+initPosition.lng+'&mode=driving&src=huiyouxing';

            androidIntent[0] = 'androidamap://route?sourceApplication=huiyouxing&slat='+myLocation.lat+'&slon='+myLocation.lng+'&sname=起点&dlat='+initPosition.lat+'&dlon='+initPosition.lng+'&dname='+positionName+'&dev=0&m=0&t=1&showType=1';
            androidIntent[1] = 'intent://map/direction?origin=latlng:'+myLocation.lat+','+myLocation.lng+'|name:起点&destination='+initPosition.lat+','+initPosition.lng+'&mode=driving&src=huiyouxing';

            window.mapCallNode=document.querySelectorAll('.callMap');

            if (typeof mapApp !== 'object') {
                mapApp = {};
            }

            (function () {
                var ua = navigator.userAgent.toLowerCase(),
                locked = false,
                domLoaded = document.readyState==='complete',
                delayToRun;

                function getAndroidVersion() {
                    var match = ua.match(/android\s([0-9\.]*)/);
                    return match ? match[1] : false;
                }

                var noIntentTest = /360 aphone|weibo|windvane/.test(ua);
                var hasIntentTest = /chrome|samsung/.test(ua);
                var isAndroid = /android|adr/.test(ua);
                var canIntent = !noIntentTest && hasIntentTest && isAndroid;
                var openInIfr = /weibo|m353/.test(ua);
                var inWeibo = ua.indexOf('weibo')>-1;

                //魅族
                if (ua.indexOf('m353')>-1 && !noIntentTest) {
                    canIntent = false;
                }

                mapApp.open = function (params, mapDom) {
                    if (!domLoaded && (ua.indexOf('360 aphone')>-1 || canIntent)) {
                        var arg = arguments;
                        delayToRun = function () {
                            AlipayWallet.open.apply(null, arg);
                            delayToRun = null;
                        };
                        return;
                    }

                    //问题来了：小米使用mapScheme,但是不能调用高德地图
                    // 通过scheme协议唤起地图
                    if (!canIntent) {
                        var mapScheme = iosScheme[params];
                        var openSchemeLink = mapDom;
                        openSchemeLink.href = mapScheme;
                        // 执行click
                        //openSchemeLink.dispatchEvent(customClickEvent());
                        

                    } else {
                        // android 下 通过 intent 协议唤起钱包
                        var mapIntent = androidIntent[params];
                        var openSchemeLink = mapDom;
                        openIntentLink.href = intentUrl;
                        // 执行click
                        //openIntentLink.dispatchEvent(customClickEvent());
                    }

                }

                if (!domLoaded) {
                    document.addEventListener('DOMContentLoaded', function () {
                        domLoaded = true;
                        if (typeof delayToRun === 'function') {
                            delayToRun();
                        }
                    }, false);
                }
            })();
        </script>
