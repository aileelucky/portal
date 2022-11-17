# 依赖地址
- [Snapshot](http://10.0.20.190:8081/#browse/browse:android_repository_snapshot:com%2Fshinemo%2Fportal)

- [Release](http://10.0.20.190:8081/#browse/browse:android_repository:com%2Fshinemo%2Fportal)

# 概述
[github查看README](https://github.com/aileelucky/portal)
- 本SDK是包含UI的项目，自带首页、门户页功能，但因为应用中心、小程序SDK等没有接入的原因，所以不包含应用组件和小程序组件；
- 首页底部菜单支持项目中扩充，门户组件也支持项目中新增；
- 目前SDK中门户组件内容比较少，数据都是mock的，集成到项目需要更新SDK版本;


# 主要类：
## PortalManiActivity
门户主页面，项目中，首页继承这个页面，就可以使用门户功能，这个页面包含了UI和首页底部菜单的逻辑。底部菜单目前就有一个门户页面，其它页面（类似"我的"、"消息"等），可以自由扩展，页面最多显示五个菜单，多的需要配置出"更多"按钮，可以把多的页面收纳。

## PortalConfig
用来配置一些初始化参数,建议在`MainActivity`初始化  
| 变量 | 描述 |必须|
|---|------------|---|
|TabConfig|首页导航配置，在这里扩展首页底部菜单，包含类型和页面|否|
|envUrl|当前环境地址:<br>切换环境要记得更新，主要用来拼接图片、文件的url<br>因为安全问题，图片的链接可能不带域名|是|
|ComponentConfig|这个类可以扩展自定义的门户组件|否|
|IAccountManager|主项目中需要实现的(一般取值都是对应`AccountManager`中的方法)|是|
|ICommonManager|用来处理一些通用的接口,项目中要实现|是|


## TabConfig
首页底部菜单扩展接口,可以参考样例项目中类[TabConfigImpl](https://git.shinemo.com/projects/YOUBAN/repos/mdos/browse/app/src/main/java/com/example/OneMDOS/TabConfigImpl.java?at=refs%2Fheads%2Fportal)
```java
    public interface TabConfig {
    
        //需要与getFragment的类型对应
        boolean isValidMenu(AppMenuVo menuVo);

        //根据配置返回Fragment
        Class<?> getFragmentClass(AppMenuVo menuVo);

        //配置底部导航需要的参数
        void setArgument(Bundle args);

        //配置菜单对应的页面，如MessageFragment,ContactFragment
        Fragment getFragment(AppMenuVo menuVo);
}
```

## ComponentConfig
门户组件扩展接口，可以实现这个接口，根据需求注入项目中新增的门户组件
```java
   public interface ComponentConfig {

    //是否需要统一调用数据源接口
    boolean needPortalContent(AppPortalElementVo elementVo);

    //ViewHolder视图
    BasePortalViewHolder getViewHolder(BaseFragment fragment, ViewGroup parent, int elementType);
}
```


## IAccountManager
项目中需要需要实现这个方法

```java
public interface IAccountManager {
    
    long getNowTime(); // 对应AccountManager的getNowTime()
   
    String getUserId(); //用户id
   
    long getCurrentOrgId(); // 当前企业id
   
    String getName();  //获取用户名
   
    String getCurrentOrgName(); //当前企业名称
}
```
## ICommonManager
通用的接口
```java
public interface ICommonManager {
    
    // CommonRedirectActivity.start()
    void commonRedirectActivityStart(Context context, String uri); 
    
    //shinemo中HomePageUtils.checkCalendarPermission()
    void checkCalendarPermission(Activity activity, List<AppPortalElementVo> mModulelist);
       
    //shinemo中HomePageUtils.getWeather()
    void getWeather(Activity activity, CompositeDisposable mCompositeSubscription, TextView tvWeather);
    
    // AppCommonUtils中的getBrandColor
    String getBrandColor(Context context);
    
    //OutsideActivity.startActivity
    void startOutSideActivity(Context context, int tab); 
}
```

## PortalMainAction
集成PortalMainActivity后，需要实现的方法，并不是全部需要实现，
| 方法 | 描述 |必须|
|---|------------|---|
|logout()|退出登录|是|
|initMessageCount()|处理消息未读数|否|
|setContactsTabDot()|通讯录提示|否|
|isForeground()|是否在前台|是|
|getDrawerFragment()|侧滑出来的“我的”页面|否|
|showContactsTip()|通讯录保护提示，参考基线`AccountUtils`中`showDialogTips`|否|

# 集成步骤
1. 添加依赖 `implementation 'com.shinemo:portal:1.0.0'`;
2. 创建`IAccountManager`和`ICommonManager`的实现类;
3. 初始化
   1. 在`Application`中调用`PortalConfig.init(Application application, IAccountManager iAccountManager, ICommonManager iCommon)`
   2. `PortalConfig.setEnvUrl("XXX")`;
   3. 如需要扩展首页菜单，创建`TabConfig`实现类 `A`，然后调用`PortalConfig.setTabConfig(new A())`;
   4. 如需要扩展门户组件，创建`ComponentConfig` 实现类 `B`,然后调用`PortalConfig.setComponentConfig(new B())`;
4. 创建MainActivity 继承PortalMainActivity;
5. 创建`PortalMainAction`的实现类,在创建MainActivity中重新方法`getPortalMainAction()`;


