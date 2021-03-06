# 哈工大冰峰实验室小型技术中台

    
为了实验室以后更快的构建业务，创建了这个小型中台

## 如何使用：
 - 将app包下的base层文件全部拷贝到新项目中
 
 - 将gradle配置中的依赖全部导入新工程即可（注意project的gradle中的allprojects中含有maven依赖
 
 - 将res包下的anim文件夹拷贝到新项目中
 
 - RxHttp即便sync成功后仍然会报错，解决方法：编译一遍即可。
 
 
## 详细的每个业务场景使用的技术构建

### 1.上拉加载，下拉刷新框架：

1.为了优化用户体验，在使用recyclerView（简称rv）时，如果不需要刷新和加载，那么就利用框架SmartRefreshLayout包裹rv，可以让布局灵活滚动（类似iOS)，提高交互体验，下面就是此纯滚动模式Layout（也就是无需上拉加载下拉刷新的模式Layout）的xml文档属性如下。

```xml
<com.scwang.smartrefresh.layout.SmartRefreshLayout
    android:id="@+id/refreshLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:srlEnablePureScrollMode="true"
    app:srlFooterMaxDragRate="5"
    app:srlReboundDuration="850">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/rv"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:overScrollMode="never" />


</com.scwang.smartrefresh.layout.SmartRefreshLayout>
```
 
 <br/>
 
2.如果需要刷新和分页上拉加载的，用如下xml代码即可

```xml

<com.scwang.smartrefresh.layout.SmartRefreshLayout
    android:id="@+id/refreshLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:srlEnableAutoLoadMore="true"
    app:srlEnableLoadMore="false"
    app:srlEnableRefresh="false"
    app:srlFooterMaxDragRate="5"
    app:srlReboundDuration="850">

    <com.scwang.smartrefresh.layout.header.ClassicsHeader
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:srlEnableLastTime="false"
        app:srlTextPulling="下拉刷新"
        app:srlTextRelease="释放刷新" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/rv"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:overScrollMode="never" />


    <com.scwang.smartrefresh.layout.footer.ClassicsFooter
        android:id="@+id/footer"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:srlFinishDuration="0" />

</com.scwang.smartrefresh.layout.SmartRefreshLayout>
```



### 2.图片相关：

1.图片加载
```Java
Glide.with(context).load(url).into(imageView);
```

2.图片预览
```Java
ImagePreviewUtil.showPreView(context, url);
```

### 3.底部弹框

```Java
BottomDialog bottomDialog = new BottomDialog().setFragmentManager(getFragmentManager())
                .setLayoutRes(R.layout.bottom_dialog_share)
                .setDimAmount(0.3f)
                .setCancelOutside(true)
                .setViewListener(dialogView -> {
                    //在这里设置你度画框中的的点击事件
                    dialogView.findViewById(R.id.xxx).setOnClickListener(xxx);
                });
        bottomDialog.show();
```

### 4.多tab页面（ViewPager+Fragment架构)

- 适配器选用：
```Java
//使用你的适配器继承此类即可，下面为使用示例
public class TestFragmentAdapter extends BaseFragmentAdapter {
    //你要展示的fragment集合，通过构造方法传入
    List<Fragment> fragments;

    public TestFragmentAdapter(@NonNull @NotNull FragmentManager fm,List<Fragment> fragments) {
        super(fm);
        this.fragments = fragments;
    }

    @NonNull
    @NotNull
    @Override
    public Fragment getItem(int position) {
        return fragments.get(position);
    }

    @Override
    public int getCount() {
        return fragments.size();
    }
}
```
<br/>

- 内部的fragment选择：

所有的fragment必须选用 
```BaseFragment``` 

其中在贤得家项目为
```
BaseCompatibleFragment
```

- fragment使用方法：


```Java

///贤得家项目替换为BaseCompatibleFragment即可
public class TestFragment extends BaseFragment {

    ///返回你的布局Xml文件ID（R.id.xxxx)
    @Override
    protected int getContentViewId() {
        return 0;
    }

    ///加载布局
    @Override
    protected void initView(View view) {

    }

    ///加载数据（这里面的逻辑自动走懒加载模式，可见才加载）
    @Override
    protected void initData() {

    }

    ///加载监听器
    @Override
    protected void initListener() {

    }

    ///点击事件（已经进行防抖动处理）
    @Override
    public void onLimitClick(View view) {

    }
}
```

### 5.适用于上拉加载，下拉刷新的页面类：

```Java
BasePagingActivity
```

- 使用示例：

```Java

/**
 * TestModel为泛型 必须填入
 */
public class TestPagingActivity extends BasePagingActivity<TestModel> {
    
    /**
    返回你的空布局Id（就是没有数据的时候，展示的页面的xmlId 
    */
    @Override
    protected int getEmptyViewId() {
        return 0;
    }

    @Override
    protected void initView() {
        /**
         *这句话在initView第一句话写，必须写
         * 作用就是初始化刷新控件，填上你的SmartRefreshLayout的id即可
         */
        initSmartRefreshLayout(R.id.layout);
        
        ///写你的逻辑
        ///xxxxx

    }

    ///初始化数据，自动调用
    @Override
    protected void initData() {


        ApiBuilder builder = new ApiBuilder()
                .Url("http//xxxx")
                .Params("page", currentPage)
        ApiClient.getInstance().doGet(builder, new CallBack<TestModel>() {
            @Override
            public void onResponse(TestModel model) {

                //设置内部适配器
                setPagingAdapter(adapter);


                //中间写你初始化的逻辑


                //最后处理完数据都要更新SmartRefreshLayout状态，
                //传入服务器返回的totalPage
                updateRefreshLayoutState(model.totalPage);

            }

            @Override
            public void onFail(String msg) {

            }
        }, TestModel.class, TestPagingActivity.this);

    }

    @Override
    protected void initListener() {

    }


    ///当刷新数据的时候会自动调用这个方法
    @Override
    protected void refreshData(int currentPage) {
        ///一般在此处调用initData()即可
    }

    ///加载 下一页数据，其中currentPage已经+1，直接用即可
    @Override
    protected void requestNextPageData(int currentPage) {

        ApiBuilder builder = new ApiBuilder()
                .Url("http//xxxx")
                .Params("page", currentPage)
        ApiClient.getInstance().doGet(builder, new CallBack<TestModel>() {
            @Override
            public void onResponse(TestModel model) {
                ///写数据更新的逻辑示例：
                ///拿到内部的PagingAdapter做更新
                getPagingAdapter().addData(model.getData());


                ///更新刷新器状态
                updateRefreshLayoutState(totolPage);
            }


            @Override
            public void onFail(String msg) {
                //如果请求失败了，要调用resetCurrentPage()
                resetCurrentPage();

            }
        }, TestModel.class, TestPagingActivity.this);
    }
    
    @Override
    public void onLimitClick(View view) {

    }
}
```


 





 
 
 
  

