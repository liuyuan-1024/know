## UIAbility概述


UIAbility是一种包含用户界面的应用组件，主要用于和用户进行交互。UIAbility也是系统调度的单元，为应用提供窗口在其中绘制界面。



每一个UIAbility实例，都对应于一个最近任务列表中的任务。



一个应用可以有一个UIAbility，也可以有多个UIAbility，如下图所示。例如浏览器应用可以通过一个UIAbility结合多页面的形式让用户进行的搜索和浏览内容；而聊天应用增加一个“外卖功能”的场景，则可以将聊天应用中“外卖功能”的内容独立为一个UIAbility，当用户打开聊天应用的“外卖功能”，查看外卖订单详情，此时有新的聊天消息，即可以通过最近任务列表切换回到聊天窗口继续进行聊天对话。



一个UIAbility可以对应于多个页面，建议将一个独立的功能模块放到一个UIAbility中，以多页面的形式呈现。例如新闻应用在浏览内容的时候，可以进行多页面的跳转使用。



## UIAbility内页面的跳转和数据传递


UIAbility的数据传递包括有UIAbility内页面的跳转和数据传递、UIAbility间的数据跳转和数据传递。



+ 在src/main/ets/entryability目录下，初始会生成一个UIAbility文件EntryAbility.ts。可以在EntryAbility.ts文件中根据业务需要实现UIAbility的生命周期回调内容。
+ 在src/main/ets/pages目录下，会生成一个Index页面。这也是基于UIAbility实现的应用的入口页面。可以在Index页面中根据业务需要实现入口页面的功能。



页面间的导航可以通过页面路由router模块来实现。页面路由模块根据页面url找到目标页面，从而实现跳转。通过页面路由模块，可以使用不同的url访问不同的页面，包括跳转到UIAbility内的指定页面、用UIAbility内的某个页面替换当前页面、返回上一页面或指定的页面等。具体使用方法请参见[ohos.router (页面路由)](https://developer.harmonyos.com/cn/docs/documentation/doc-references-V3/js-apis-router-0000001478061893-V3?catalogVersion=V3)。



### 页面跳转和参数接收


在使用页面路由之前，需要先导入router模块，如下代码所示。



```arkts
import router from '@ohos.router';
```



页面跳转的几种方式，根据需要选择一种方式跳转即可。



API9及以上，router.pushUrl()方法新增了mode参数，可以将mode参数配置为router.RouterMode.Single单实例模式和router.RouterMode.Standard多实例模式。



+ 方式一：在单实例模式下：如果目标页面的url在页面栈中已经存在同url页面，离栈顶最近同url页面会被移动到栈顶，移动后的页面为新建页，原来的页面仍然存在栈中，页面栈的元素数量不变；如果目标页面的url在页面栈中不存在同url页面，按多实例模式跳转，页面栈的元素数量会加1。



```arkts
router.pushUrl({
  url: 'pages/Second',
  params: {
	src: 'Index页面传来的数据',
  }
}, router.RouterMode.Single)
```



+ 方式二：在单实例模式下：如果目标页面的url在页面栈中已经存在同url页面，离栈顶最近同url页面会被移动到栈顶，替换当前页面，并销毁被替换的当前页面，移动后的页面为新建页，页面栈的元素数量会减1；如果目标页面的url在页面栈中不存在同url页面，按多实例模式跳转，页面栈的元素数量不变。



```arkts
router.replaceUrl({
	url: 'pages/Second',
	params: {
		src: 'Index页面传来的数据',
	}
}, router.RouterMode.Single)
```



通过调用router.getParams()方法获取Index页面传递过来的自定义参数。



```arkts
import router from '@ohos.router';

@Entry
@Component
struct Second {
  @State src: string = (router.getParams() as Record<string, string>)['src'];
  // 页面刷新展示
  // ...
}
```



### 页面返回和参数接收


可以通过调用router.back()方法实现返回到上一个页面，或者在调用router.back()方法时增加可选的options参数（增加url参数）返回到指定页面。



+ 调用router.back()返回的目标页面需要在页面栈中存在才能正常跳转。
+ 例如调用router.pushUrl()方法跳转到Second页面，在Second页面可以通过调用router.back()方法返回到上一个页面。
+ 例如调用router.clear()方法清空了页面栈中所有历史页面，仅保留当前页面，此时则无法通过调用router.back()方法返回到上一个页面。



```arkts
// 返回上一个页面
router.back();

// 返回到指定页面
router.back({
  url: 'pages/Index',
  params: {
    src: '返回时的自定义参数',
  }
});

// 页面返回可以根据业务需要增加一个询问对话框。即在调用router.back()方法之前，可以先调用router.enableBackPageAlert()方法开启页面返回询问对话框功能。

router.enableBackPageAlert({
  message: 'Message Info'
});

router.back();
```



+ router.enableBackPageAlert()方法开启页面返回询问对话框功能，只针对当前页面生效。例如在调用router.pushUrl()或者router.replaceUrl()方法，跳转后的页面均为新建页面，因此在页面返回之前均需要先调用router.enableBackPageAlert()方法之后，页面返回询问对话框功能才会生效。
+ 如需关闭页面返回询问对话框功能，可以通过调用router.disableAlertBeforeBackPage()方法关闭该功能即可。



调用router.back()方法，不会新建页面，返回的是原来的页面，在原来页面中@State声明的变量不会重复声明，以及也不会触发页面的aboutToAppear()生命周期回调，因此无法直接在变量声明以及页面的aboutToAppear()生命周期回调中接收和解析router.back()传递过来的自定义参数。



可以放在业务需要的位置进行参数解析。比如在在Index页面中的onPageShow()生命周期回调中进行参数的解析。



```arkts
import router from '@ohos.router';
class routerParams {
  src:string
  constructor(str:string) {
    this.src = str
  }
}
@Entry
@Component
struct Index {
  @State src: string = '';
  onPageShow() {
    this.src = (router.getParams() as routerParams).src
  }
  // 页面刷新展示
  // ...
}
```



## UIAbility的生命周期


当用户浏览、切换和返回到对应应用的时候，应用中的UIAbility实例会在其生命周期的不同状态之间转换。



UIAbility类提供了很多回调，通过这些回调可以知晓当前UIAbility的某个状态已经发生改变：例如UIAbility的创建和销毁，或者UIAbility发生了前后台的状态切换。



在UIAbility的使用过程中，会有多种生命周期状态。

<!-- 这是一张图片，ocr 内容为： -->
![](assets/Pasted%20image%2020240115004626.png)<!-- 这是一张图片，ocr 内容为：UIABILITY START CREATE WINDOWSTAGECREATE FOREGROUND BACKGROUND WINDOWSTAGEDESTROY DESTROY UIABILITY END -->
![](https://cdn.nlark.com/yuque/0/2024/png/43050341/1710503796499-478481d4-13b1-4745-bf1a-fbfd47bca8b2.png)



每一个生命周期函数都对一个名为 **onXxxx** 的回调函数



1. Create状态，在UIAbility实例创建时触发，系统会调用onCreate回调。可以在onCreate回调中进行相关初始化操作。



```arkts
import UIAbility from '@ohos.app.ability.UIAbility';
import window from '@ohos.window';

export default class EntryAbility extends UIAbility {
	onCreate(want: Want, launchParam: AbilityConstant.LaunchParam) {
		// 应用初始化
		// ...
	}
	// ...
}
```



2. UIAbility实例创建完成之后，在进入Foreground之前，系统会创建一个WindowStage。每一个UIAbility实例都对应持有一个WindowStage实例。  
WindowStage为本地窗口管理器，用于管理窗口相关的内容，例如与界面相关的获焦/失焦、可见/不可见。  
可以在onWindowStageCreate回调中，设置UI页面加载、设置WindowStage的事件订阅。  
在onWindowStageCreate(windowStage)中通过loadContent接口设置应用要加载的页面。  
Window接口的使用详见[窗口开发指导](https://developer.harmonyos.com/cn/docs/documentation/doc-guides-V3/application-window-stage-0000001427584712-V3?catalogVersion=V3)。



```arkts
import UIAbility from '@ohos.app.ability.UIAbility';
import window from '@ohos.window';

export default class EntryAbility extends UIAbility {
    // ...

    onWindowStageCreate(windowStage: window.WindowStage) {
        // 设置UI页面加载
        // 设置WindowStage的事件订阅（获焦/失焦、可见/不可见）
        // ...

        windowStage.loadContent('pages/Index', (err, data) => {
            // ...
        });
    }
    // ...
}
```



3. Foreground和Background状态，分别在UIAbility切换至前台或者切换至后台时触发。分别对应于onForeground回调和onBackground回调。  
onForeground回调，在UIAbility的UI页面可见之前，即UIAbility切换至前台时触发。可以在onForeground回调中申请系统需要的资源，或者重新申请在onBackground中释放的资源。  
onBackground回调，在UIAbility的UI页面完全不可见之后，即UIAbility切换至后台时候触发。可以在onBackground回调中释放UI页面不可见时无用的资源，或者在此回调中执行较为耗时的操作，例如状态保存等。



```arkts
import UIAbility from '@ohos.app.ability.UIAbility';
import window from '@ohos.window';

export default class EntryAbility extends UIAbility {
    // ...

    onForeground() {
        // 申请系统需要的资源，或者重新申请在onBackground中释放的资源
        // ...
    }

    onBackground() {
        // 释放UI页面不可见时无用的资源，或者在此回调中执行较为耗时的操作
        // 例如状态保存等
        // ...
    }
}
```



4. 在UIAbility实例销毁之前，则会先进入onWindowStageDestroy回调，我们可以在该回调中释放UI页面资源。



```arkts
import UIAbility from '@ohos.app.ability.UIAbility';
import window from '@ohos.window';

export default class EntryAbility extends UIAbility {
    // ...

    onWindowStageDestroy() {
        // 释放UI页面资源
        // ...
    }
}
```



5. Destroy状态，在UIAbility销毁时触发。可以在onDestroy回调中进行系统资源的释放、数据的保存等操作。



```arkts
import UIAbility from '@ohos.app.ability.UIAbility';
import window from '@ohos.window';

export default class EntryAbility extends UIAbility {
    // ...

    onDestroy() {
        // 系统资源的释放、数据的保存等
        // ...
    }
}
```



## UIAbility的启动模式


对于浏览器或者新闻等应用，用户在打开该应用，并浏览访问相关内容后，回到桌面，再次打开该应用，显示的仍然是用户当前访问的界面。



对于应用的分屏操作，用户希望使用两个不同应用（例如备忘录应用和图库应用）之间进行分屏，也希望能使用同一个应用（例如备忘录应用自身）进行分屏。



对于文档应用，用户从文档应用中打开一个文档内容，回到文档应用，继续打开同一个文档，希望打开的还是同一个文档内容。



基于以上场景的考虑，UIAbility当前支持singleton（单实例模式）、multiton（多实例模式）和specified（指定实例模式）3种启动模式。



启动模式的开发使用，在 **module.json5** 文件中的 **“launchType”** 字段配置为 **“singleton/multiton/specified”** 即可。



```arkts
{
   "module": {
     // ...
     "abilities": [
       {
         "launchType": "singleton",
         // ...
       }
     ]
  }
}
```



对启动模式的详细说明如下：



+ singleton（单实例模式）  
singleton启动模式为单实例模式，也是默认情况下的启动模式。  
每次调用startAbility()方法时，如果应用进程中该类型的UIAbility实例已经存在，则复用系统中的UIAbility实例。系统中只存在唯一一个该UIAbility实例，即在最近任务列表中只存在一个该类型的UIAbility实例。  
应用的UIAbility实例已创建，该UIAbility配置为单实例模式，再次调用startAbility()方法启动该UIAbility实例。由于启动的还是原来的UIAbility实例，并未重新创建一个新的UIAbility实例，此时只会进入该UIAbility的onNewWant()回调，不会进入其onCreate()和onWindowStageCreate()生命周期回调。
+ multiton（多实例模式）  
multiton启动模式为多实例模式，每次调用startAbility()方法时，都会在应用进程中创建一个新的该类型UIAbility实例。即在最近任务列表中可以看到有多个该类型的UIAbility实例。这种情况下可以将UIAbility配置为multiton（多实例模式）。
+ specified（指定实例模式）  
specified启动模式为指定实例模式，针对一些特殊场景使用（例如文档应用中每次新建文档希望都能新建一个文档实例，重复打开一个已保存的文档希望打开的都是同一个文档实例）。 
    1. 当应用的UIAbility实例已经被创建，并且配置为指定实例模式时，如果再次调用startAbility()方法启动该UIAbility实例，且AbilityStage的onAcceptWant()回调匹配到一个已创建的UIAbility实例，则系统会启动原来的UIAbility实例，并且不会重新创建一个新的UIAbility实例。此时，该UIAbility实例的onNewWant()回调会被触发，而不会触发onCreate()和onWindowStageCreate()生命周期回调。
    2. DevEco Studio默认工程中未自动生成AbilityStage，AbilityStage文件的创建请参见AbilityStage组件容器。



每一个UIAbility中包含一个Context属性

