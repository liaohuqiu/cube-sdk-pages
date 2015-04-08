---
layout: default
title: 加载更多
lead: "简单，易用，可扩展"
---

<h1 id='feature'>加载更多简介</h1>
---

加载更多是应用常用的一种交互形式。经常会和下拉刷新一起出现。不过这两个是不同的组件。

本方案仅关注加载更多。结构简单，使用方便。可扩展性极强。

### 应用场景

ListView 或者 GridView 滑动到底部时。自动加载更多或者手动点击加载更多。

加载失败后，点击加载更多。

根据应用情况，可以一进入页面就触发加载更多接口，加载第一页，也可不触发。

### 目前支持的控件

1. ListView
2. GridViewWithHeaderAndFooter

    普通的GridView无法支持addHeaderView 和 addFooterView。`GridViewWithHeaderAndFooter` 支持。

### 特性

1. 自动加载更多 / 点击加载更多
2. 错误时，点击加载重试
3. 轻松自定义UI自定义UI

<h1 id='how-to'>用法</h1>
---

### xml 中布局好元素

1. ListView

    ```xml
    <in.srain.cube.views.loadmore.LoadMoreListViewContainer
        android:id="@+id/load_more_list_view_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/cube_mints_white">

        <ListView
            android:id="@+id/load_more_small_image_list_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:divider="@null"
            android:fadingEdge="none"
            android:listSelector="@android:color/transparent"
            android:paddingLeft="12dp"
            android:paddingRight="12dp"
            android:scrollbarStyle="outsideOverlay" />

    </in.srain.cube.views.loadmore.LoadMoreListViewContainer>
    ```

1. GridViewWithHeaderAndFooter

    ```xml
    <in.srain.cube.views.loadmore.LoadMoreGridViewContainer
        android:id="@+id/load_more_grid_view_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/cube_mints_white">
    
        <in.srain.cube.views.GridViewWithHeaderAndFooter
            android:id="@+id/load_more_grid_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fadingEdge="none"
            android:focusable="false"
            android:horizontalSpacing="10dp"
            android:listSelector="@android:color/transparent"
            android:numColumns="2"
            android:paddingLeft="12dp"
            android:paddingRight="12dp"
            android:scrollbarStyle="outsideOverlay"
            android:stretchMode="columnWidth" />
    
    </in.srain.cube.views.loadmore.LoadMoreGridViewContainer>
    ```

### 代码中配置

1. findViewById, 配置

    ```java
    // 获取view的引用
    final LoadMoreListViewContainer loadMoreListViewContainer = (LoadMoreListViewContainer) view.findViewById(R.id.load_more_list_view_container);
    
    // 使用默认样式
    loadMoreListViewContainer.useDefaultHeader();
    ```

2.  加载更多数据

    当列表滑动到最底部时，触发加载更多操作，这时你需要从网络加载数据或者从数据去读取:

    ```java
    loadMoreListViewContainer.setLoadMoreHandler(new LoadMoreHandler() {
        @Override
        public void onLoadMore(LoadMoreContainer loadMoreContainer) {
            // 请求下一页数据
            mDataModel.queryNextPage();
        }
    });
    ```

3. 加载完成之后，设置状态
    * 是否内容为空
    * 是否有更多内容

    ```
    // 数据加载完后，设置是否为空，是否有更多
    loadMoreListViewContainer.loadMoreFinish(mDataModel.getListPageInfo().isEmpty(), mDataModel.getListPageInfo().hasMore());
    
    // 更新列表显示
    mAdapter.notifyDataSetChanged();
    ```

4. 加载失败处理

    ```java
    int errorCode = 0;
    String errorMessage = "加载失败，点击加载更多";
    loadMoreListViewContainer.loadMoreError(errorCode, errorMessage);
    ```

<h1 id='customize'>自定义UI</h1>
---

默认样式实在是太简单了，甚至简陋了。各个应用都有自己丰富的UI表现形式。

目前设计可以很方便地定制UI样式:

### 接口定义

默认样式支持几种状态显示:

1. 加载中
2. 加载完毕，数据为空
3. 加载完毕，没有更多数据
4. 点击加载更多
5. 加载失败

对于以上几种状态，抽象出接口, `LoadMoreUIHandler`, 处理加载时各种状态:

```java
public interface LoadMoreUIHandler {

    // 加载中
    public void onLoading(LoadMoreContainer container);

    // 加载完成: 数据为空 / 没有更多
    public void onLoadFinish(LoadMoreContainer container, boolean empty, boolean hasMore);

    // 等待加载更多
    public void onWaitToLoadMore(LoadMoreContainer container);

    // 加载失败
    public void onLoadError(LoadMoreContainer container, int errorCode, String errorMessage);
}
```

### 接口实现

实现该接口之后, 比如有一个类实现了此接口，负责各种状态显示, `ISLoadMoreFooterView`, 如下:

```java
public class ISLoadMoreFooterView extends RelativeLayout 
        implements LoadMoreUIHandler {

}
```

### 使用

```java
// 创建
ISLoadMoreFooterView footerView = new ISLoadMoreFooterView(container.getContext());

// 根据实际情况设置样式
AbsListView.LayoutParams lp = new AbsListView.LayoutParams(-2, LocalDisplay.dp2px(80));
footerView.setLayoutParams(lp);

// 设置
container.setLoadMoreView(footerView);
container.setLoadMoreUIHandler(footerView);
```
