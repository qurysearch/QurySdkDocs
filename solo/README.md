# QurySDK接入文档

版本号|更新内容
:-:|:-:
1.0.3|删除设置GoogleAdId接口，修复已知bug
1.0.1|联调第一版

## 一、获取ApiKey

ApiKey是每个使用者独有的标识，也是使用 QurySDK 的必要元素，接入前请联系对接人员获取ApiKey，在初始化SDK时传入即可。


## 二、Android Studio 工程配置

### 2.1 环境要求
JDK8+

### 2.2 build.gradle 配置
在 `Project` 的 `build.gradle` 文件中添加依赖库仓库地址：

```allprojects {
    repositories {
        ......
        
        maven {
            url "https://raw.githubusercontent.com/qurysearch/QurySdk/master/"
        }
} }
```

在 `App Module` 的 `build.gradle` 文件中添加一下依赖：

```dependencies {
    ......
    
    implementation 'com.qury:solo:1.0.2'
}
```

### 2.3 布局文件配置
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/white">
    
    ······

    <com.qury.sdk.ui.view.QurySearchResultView
        android:id="@+id/qurySearchResultView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_below="@+id/search_widget"
        android:layout_alignParentBottom="true"/>
</RelativeLayout>
```

### 2.4 SDK中使用的第三方库
```
//RecycleView
api 'com.google.android.material:material:1.0.0'
//OkHttp3
api 'com.squareup.okhttp3:okhttp:3.14.1'
api 'com.squareup.okhttp3:okhttp-urlconnection:3.14.1'
//GSon
api 'com.google.code.gson:gson:2.8.5'
//StatusBar
api 'com.githang:status-bar-compat:0.7'
//ChromeTab
api 'androidx.browser:browser:1.0.0'
//flurry
api 'com.flurry.android:analytics:11.6.0@aar'
// For google play service
api 'com.google.android.gms:play-services-ads:6.5.87'
```


## 三、功能介绍

### 3.1 初始化和关闭 SDK
初始化 SDK 时按照如下方式进行初始化：

```
QuryConfig config = new QuryConfig.Builder(this)
                .setApiKey(API_KEY)
                .setApiChannel(API_CHANNEL)
                .setImageLoader(new MyImageLoader())
                .setClickListener(new MyClickListener())
                .build();
QurySDK.initialize(config);
```

关闭 SDK 时按照如下方式进行关闭：

```
QurySDK.shutdown();
```
> 注意：一次使用周期内多次调用时只有第一次生效，直到 shutdown 方法被调用；关闭 SDK 后必须再次初始化才可以使用。

### 3.2 夜间模式
通过如下设置可以开启夜间模式

```
searchResultView.setNightMode(true);
```

### 3.3 横屏模式
通过如下设置可以开启横屏模式

```
searchResultView.setLandscape(true);
```

### 3.4 搜索推荐
QurySDK 包含搜索推荐功能，当 QurySearchResultView 被加载到页面时或清空搜索条件时自动展示搜索推荐内容：
![pdf-1](../img/pdf-1.jpeg)
> 如果不需要展示搜索推荐内容，请联系对接人员关闭搜索推荐功能

### 3.5 搜索直达
执行如下代码展示搜索结果：

```
searchResultView.query("dress", new AsyncQueryListener() {

	    @Override
	    public void onPreQuery() {
	    	···
	    }
	
	    @Override
	    public void onDealResult(boolean isSuccess) {
	    	···
	    }
});
```
![pdf-2](../img/pdf-2.jpeg)

### 3.6 固定的头部和脚部
当搜索直达结果的头部或脚部需要展示自己的 View，且 View 不会随着搜索条件变化，可以通过下面的接口实现：

```
searchResultView.addFixedHeader(new ViewHolderWrapper());
//或
searchResultView.addFixedFooter(new ViewHolderWrapper());
```
> ViewHolderWrapper 需要自定义类继承后实现里面的接口

取消头部或脚部的自定义 View 时，传入null即可

```
searchResultView.addFixedHeader(null);
//或
searchResultView.addFixedFooter(null);
```
> 头部和脚部不会同时存在，同时设置只有第一个会生效

### 3.7 动态的头部和脚部
当搜索直达结果头部或脚部的 View 需要随搜索条件动态展示，可以通过下面的接口实现：

```
searchResultView.addDynamicHeader(new ViewHolderWrapper(), searchKey);
//或
searchResultView.addDynamicFooter(new ViewHolderWrapper(), searchKey);
```
> ViewHolderWrapper 需要自定义类继承后实现里面的接口

取消头部或脚部的自定义 View 时，传入null即可

```
searchResultView.addDynamicHeader(null, searchKey);
//或
searchResultView.addDynamicFooter(null, searchKey);
```
> 头部和脚部不会同时存在，同时设置只有第一个会生效

### 3.8 头部和脚部列表
当搜索直达结果头部或脚部需要展示不止一个 View，而是一个列表时，可以通过下面的接口实现：

```
searchResultView.setHeaderAdapter(adapter);
searchResultView.setHeaderData(datalist, searchKey);
//或
searchResultView.setFooterAdapter(adapter);
searchResultView.setFooterData(datalist, searchKey);
```

取消头部或脚部列表时，传入null即可

```
searchResultView.setHeaderAdapter(null);
//或
searchResultView.setFooterAdapter(null);
```

当仅清空头部或脚部列表时，如下操作即可：

```
searchResultView.setHeaderData(null, null);
//或
searchResultView.setFooterData(null, null);
```
> 头部和脚部不会同时存在，同时设置只有第一个会生效


## 四、API介绍

### 4.1 QurySDK

```
/**
 * 初始化SDK，一次使用周期内多次调用时只有第一次生效，直到 shutdown 方法被调用。
 * @param config SDK配置类，参见类 QueryConfig.Builder
 */
public static void initialize(QuryConfig config);

/**
 * 关闭SDK，SDK使用结束后一定要调用此方法，以免发生内存泄漏。
 */
public static void shutdown();
```

### 4.2 QuryConfig.Builder

```
/**
 * 构造方法
 * @param context 全局上下文
 */
public Builder(Context context);

/**
 * 配置 Api Key
 * @param apiKey SDK使用者标识，联系对接人员获取
 */
public Builder setApiKey(String apiKey);

/**
 * 配置 Api Channel
 * @param apiChannel SDK使用者自定义标识
 */
public Builder setApiChannel(String apiChannel);

/**
 * 配置图片加载回调接口
 * @param imageLoader 图片加载回调，参见接口 GeneralImageLoader
 */
public Builder setImageLoader(GeneralImageLoader imageLoader);

/**
 * 配置搜索结果点击回调
 * @param listener 点击回调，参见接口 QuryClickListener
 */
public Builder setClickListener(QuryClickListener listener);

/**
 * 构建QuryConfig对象
 */
public QuryConfig build();
```

### 4.3 QuryField

```
/**
 * 获取GoogleAdId
 * @param context 上下文
 * @param observer GoogleAdId回调接口
 */
public static void readGoogleAdId(Context context, FieldObserver observer);
```

### 4.4 FieldObserver

```
/**
 * GoogleAdId 回调
 * @param context   上下文
 * @param adId      GoogleAdId
 */
public void onGoogleAdId(Context context, String adId);
```


### 4.5 GeneralImageLoader

```
/**
 * 加载图片回调方法
 *
 * @param imageView 展示图片View
 * @param uri       图片Uri地址
 * @param placeId   图片加载中占位图
 * @param errorId   图片加载失败占位图
 */
public void load(ImageView imageView, Uri uri, @DrawableRes int placeId, @DrawableRes int errorId);

/**
 * 加载图片回调方法
 *
 * @param imageView 展示图片View
 * @param url       图片url地址
 * @param placeId   图片加载中占位图
 * @param errorId   图片加载失败占位图
 */
public void load(ImageView imageView, String url, @DrawableRes int placeId, @DrawableRes int errorId);

/**
 * 加载图片回调方法
 *
 * @param imageView 展示图片View
 * @param resId     图片资源id
 * @param placeId   图片加载中占位图
 * @param errorId   图片加载失败占位图
 */
public void load(ImageView imageView, @DrawableRes int resId, @DrawableRes int placeId, @DrawableRes int errorId);
```

### 4.6 QuryClickListener

```
/**
 * 点击事件来源：点击搜索直达结果
 */
int FROM_ORGANIC = 11;

/**
 * 点击事件来源：点击搜索直达结果的Item
 */
int FROM_ORGANIC_ITEM = 12;

/**
 * 点击事件来源：点击App下载广告
 */
int FROM_AD_APP = 21;

/**
 * 点击事件来源：点击网页广告
 */
int FROM_AD_WEB = 22;

/**
 * 点击事件来源：点击搜索推荐广告
 */
int FROM_AD_TILES = 23;
    
/**
 * 打开第三方应用
 *
 * @param context	上下文
 * @param packageId	应用包名
 * @param url 		用于打开App的url
 * @param source 	点击事件来源，参照上面点击事件来源
 * @return			是否消费点击事件，true:消费，false:不消费
 */
public boolean onTurnToApp(Context context, String packageId, String url, int source);

/**
 * 打开网页
 *
 * @param context	上下文
 * @param title		网页标题
 * @param url 		网页地址
 * @param source 	点击事件来源，参照上面点击事件来源
 * @return			是否消费点击事件，true:消费，false:不消费
 */
public boolean onTurnToWeb(Context context, String title, String url, int source);
```

### 4.7 QurySearchResultView

```
/**
 * 搜索接口
 *
 * @param searchKeyWord 搜索条件
 */
public final void query(@NonNull String searchKeyWord);

/**
 * 搜索接口，带回调
 *
 * @param searchKeyWord 搜索条件
 * @param listener      搜索回调，参见接口 AsyncQueryListener
 */
public final void query(@NonNull String searchKeyWord, @NonNull AsyncQueryListener listener);

/**
 * 清空搜索条件和搜索结果
 */
public final void emptyView();

/**
 * 过时方法
 * 显示Web搜索结果
 * @param searchKeyWord 搜索条件
 */
@Deprecated
public final void queryWithWebMode(@NonNull String searchKeyWord);

/**
 * 过时方法
 * 显示Web搜索结果
 * @param searchKeyWord 搜索条件
 * @param listener      搜索回调，参见接口 AsyncQueryListener
 */
@Deprecated
public final void queryWithWebMode(@NonNull String searchKeyWord, @NonNull AsyncQueryListener listener);

/**
 * 返回Qury结果数量
 *
 * @return 
 */
public final int getQuryItemCount();

/**
 * 配置显示模式，true:夜间模式，false:日间模式
 * 默认为日间模式
 * @param nightMode 显示模式
 */
public final void setNightMode(boolean nightMode);

/**
 * 配置横屏模式，true:横屏模式，false:竖屏模式
 * 默认为竖屏
 *
 * @param landscap
 */
public final void setLandscape(boolean landscap);

/**
 * 配置搜索结果点击回调
 * 该方法和 QuryConfig.Builder#setClickListener 方法效果相同，会覆盖之前的配置。
 * @param listener 点击回调，参见接口 QuryClickListener
 */
public final void setQuryClickListener(QuryClickListener listener);
    
/**
 * 添加固定的头部 View
 * 与其他添加头部或脚部方法互斥
 * @param customWrapper 参见类 ViewHolderWrapper
 */
public void addFixedHeader(ViewHolderWrapper customWrapper);

/**
 * 添加固定的脚部 View
 * 与其他添加头部或脚部方法互斥
 * @param customWrapper 参见类 ViewHolderWrapper
 */
public void addFixedFooter(ViewHolderWrapper customWrapper);

/**
 * 添加动态的头部 View
 * 与其他添加头部或脚部方法互斥
 * @param customWrapper 参见类 ViewHolderWrapper
 * @param matchKey 匹配条件，等于搜索条件时才展示动态头部 View
 */
public void addDynamicHeader(ViewHolderWrapper customWrapper, String matchKey);

/**
 * 添加动态的脚部 View
 * 与其他添加头部或脚部方法互斥
 * @param customWrapper 参见类 ViewHolderWrapper
 * @param matchKey 匹配条件，等于搜索条件时才展示动态脚部 View
 */
public void addDynamicFooter(ViewHolderWrapper customWrapper, String matchKey);

/**
 * 设置头部列表 Adapter
 * @param adapter 
 */
public void setHeaderAdapter(RecyclerView.Adapter adapter);

/**
 * 设置脚部列表 Adapter
 * @param adapter 
 */
public void setFooterAdapter(RecyclerView.Adapter adapter);

/**
 * 设置头部列表数据
 * @param list 列表数据，参见类HeaderDataItem
 * @param matchKey 匹配条件，等于搜索条件时才展示头部列表
 */
public void setHeaderData(List<? extends HeaderDataItem> list, String matchKey);

/**
 * 设置脚部列表数据
 * @param list 列表数据，参见类 FooterDataItem
 * @param matchKey 匹配条件，等于搜索条件时才展示脚部列表
 */
public void setFooterData(List<? extends FooterDataItem> list, String matchKey);
```

### 4.8 ViewHolderWrapper

```
/**
 * 初始化ViewHolder
 * @param parent 
 * @param viewType
 */
@NonNull
public abstract RecyclerView.ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType);

/**
 * 绑定ViewHolder
 * @param holder 
 * @param position
 */
public abstract void onBindViewHolder(@NonNull RecyclerView.ViewHolder holder, int position);
```

### 4.9 HeaderDataItem

头部列表数据基类

### 4.10 FooterDataItem

脚部列表数据基类


## 五、ProGuard 混淆
混淆规则文件已⾃动打⼊依赖库，外部调用者⽆需额外配置。