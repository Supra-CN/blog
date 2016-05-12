title: Staggered layout study
speaker: 王佳
url: localhost:8000

[slide]
# staggered layout

是一种支持每行元素视图拥有不同高度的Gird layout.

![img-stsy][]
![img-maurycyw][]

[img-stsy]: http://f.cl.ly/items/2z1B0Y0M0G0O2k1l3J03/Trending.png
[img-maurycyw]: http://f.cl.ly/items/1I0n3i361o3R070y3k46/340616_1355789751.jpeg

[slide]
# 1 实现staggered layout的主流方案 {:&.flexbox.vleft}
 * [android/StaggeredGridLayoutManager][site-android]
 * [etsy/AndroidStaggeredGrid][site-etsy]
 * [maurycyw/StaggeredGridView][site-maurycyw]
 * [GDG-Korea/PinterestLikeAdapterView][site-GDG-Korea]

[site-android]: http://developer.android.com/reference/android/support/v7/widget/StaggeredGridLayoutManager.html
[site-etsy]: https://github.com/etsy/AndroidStaggeredGrid
[site-maurycyw]: https://github.com/maurycyw/StaggeredGridView
[site-GDG-Korea]: https://github.com/GDG-Korea/PinterestLikeAdapterView

[slide]
## 1.1 android/StaggeredGridLayoutManager
 * 许可：Apache-2.0 {:&.rollIn}
 * 特性：采用RecyclerView以LayoutManager的形式创建和管理CardView
 * 优点：android框架官方统一的卡片内容展示方案、稳定、强大、易于维护
 * 限制：需要引入android.support.v7包
 * 选型：具有统一的技术方案，适用于绝大多数的场景，首选方案

[slide]
## 1.2 etsy/AndroidStaggeredGrid  
 * 许可：Apache-2.0 {:&.rollIn}
 * 特性：etsy公司因在实现其Etsy app时发现Google的Android库中没有提供相关功能以实现其需求而开发，继承自AbsListView 
 * 优点：稳定，支持页眉页脚，支持模、竖屏都保持单元格显示同步，github star 4k+ fork 1k+
 * 限制：作者表示不再维护该项目，并推荐采用StaggeredGridLayoutManager
 * 选型：第三方主流方案，但是已经停止维护了

[slide]
## 1.3 maurycyw/StaggeredGridView
 * 许可：无限制 {:&.rollIn}
 * 特性：这是android初期尝试的StaggeredGridView的一个修改版本，继承自ViewGroup
 * 优点：短小精干，支持于GridView相同的部分xml属性，学习成本低，使用方便
 * 限制：每个卡片item必须定死高度尺寸，不能使用自适应属性，否则会引起视图行为异常
 * 选型：适合用于快速开发简单的需求

[slide]
## 1.4 GDG-Korea/PinterestLikeAdapterView
 * 许可：Apache-2.0 {:&.rollIn}
 * 特性：继承自基于ViewGroup重新封装的ListView，棒子出品
 * 优点：重写的代码很多，二次开发比较灵活
 * 限制：二次开发太多，可能不稳定，不支持xml配置、Choice Mode、Filter、Handle Key Event 等等等
 * 选型：看起来也不错，但是二次开发太多让人信不过其稳定性，可以作为etsy/AndroidStaggeredGrid停止维护后的备选方案

[slide]
# 2 方案选型 {:&.flexbox.vleft}
以上4个实现方案都采用了Apache-2.0以及更宽松的许可方式，都可以在商业项目里放心使用；

 * 在需求比较简单且开发工时和包大小有要求的情况下，可以采用maurycyw/StaggeredGridView快速实现 {:&.rollIn}
 * 在项目没有特殊要求的情况下推荐首选android/StaggeredGridLayoutManager方案；
 * 有特殊原因不能使用android/StaggeredGridLayoutManager时，建议采用etsy/AndroidStaggeredGrid方案；
 * 如果etsy/AndroidStaggeredGrid遇到重大问题时，采用GDG-Korea/PinterestLikeAdapterView方案；

[slide]
# 3 主流第三方实现方案分析 {:&.flexbox.vleft}
抱着好奇宝宝的心态，分析一下主流第三方实现方案： `etsy/AndroidStaggeredGrid`   

方便起见，以下简称为: `EtsyStaggered`  

[slide]
## 3.1 配置 EtsyStaggered
EtsyStaggered在maven中心仓库可用，在工程中添加如下依赖即可；
```groovy
repositories {
    mavenCentral()
}

dependencies {
    compile 'com.etsy.android.grid:library:x.x.x' // see changelog
}
```

[slide]
## 3.2 EtsyStaggered的基本使用
### 3.2.1 在布局文件中声明StaggeredGridView.
```xml
    <com.etsy.android.grid.StaggeredGridView
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:id="@+id/grid_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:item_margin="8dp"
        app:column_count="@integer/column_count" />
```

[slide]
### 3.2.2 配置视图的属性.
 * `item_margin` - item周围的margin (默认 0dp).
 * `column_count` - 展示的列数. 这将会覆盖已有的column_count_portrait和column_count_landscape属性 (默认 0)
 * `column_count_portrait` - 纵向显示的列数 (默认 2).
 * `column_count_landscape` - 横向显示的列数 (默认 3).
 * `grid_paddingLeft` - item左边留白. 对headers/footers不生效 (默认 0).
 * `grid_paddingRight` - item右边留白. 对headers/footers不生效 (默认 0).
 * `grid_paddingTop` - item顶部留白. 对headers/footers不生效 (默认 0).
 * `grid_paddingBottom` - item底部留白. 对headers/footers不生效 (默认 0).

[slide]
### 3.2.3 用GridView/ListView相同的方法设置adapter.
```java
ListAdapter adapter = ...;

StaggeredGridView gridView = (StaggeredGridView) findViewById(R.id.grid_view);

gridView.setAdapter(adapter);
```

[slide]
## 3.3 代码实现分析
```
AndroidStaggeredGrid/library/src/main$ tree
├ AndroidManifest.xml
├ java
│ └ com
│   └ etsy
│     └ android
│       └ grid
│         ├ ClassLoaderSavedState.java    //可以保存类集成关系连上所有状态
│         ├ ExtendableListView.java       //为了支持展开效果而重写的ExtendableListView实现
│         ├ HeaderViewListAdapter.java    //支持header和fooder的WrapperListAdapter
│         ├ StaggeredGridView.java        //交错布局自定义视图
│         └ util
│           ├ DynamicHeightImageView.java //一种通过宽高比自动来动态维护视图高度的ImageView
│           └ DynamicHeightTextView.java  //一种通过宽高比自动来动态维护视图高度的TextView
└ r
  └ values
    └ attrs.xml                           //自定义视图属性
    
    8 directories, 8 files
```

[slide]
### 3.3.1 EtsyStaggered核心布局算法分析.
**StaggeredGridView几个重要的成员变量**
```
private int mColumnCount; // 瀑布流的列数
private int mColumnWidth; // 程序瀑布流的列宽度
private int mColumnCountPortrait = DEFAULT_COLUMNS_PORTRAIT; // 瀑布流竖屏显示的列数
private int mColumnCountLandscape = DEFAULT_COLUMNS_LANDSCAPE;// 瀑布流横屏现实的列数
private int[] mColumnTops;  // 每列最顶端item的位置
private int[] mColumnBottoms; // 每列最底端item的位置
private int[] mColumnLefts;  // 每列最左端item的位置
```

[slide]
### 3.3.1 EtsyStaggered核心布局算法分析.
**StaggeredGridView计算每列的宽度**
```
@Override
	protected void onMeasure(final int widthMeasureSpec, final int heightMeasureSpec) {
		super.onMeasure(widthMeasureSpec, heightMeasureSpec);
		if (mColumnCount <= 0) { 
			boolean isLandscape = isLandscape();
			mColumnCount = isLandscape ? mColumnCountLandscape : mColumnCountPortrait;
		}
		// our column width is the width of the listview 
		// minus it's padding
		// minus the total items margin
		// divided by the number of columns
		mColumnWidth = calculateColumnWidth(getMeasuredWidth());
	    ...
    }
    
 private int calculateColumnWidth(final int gridWidth) {
        final int listPadding = getRowPaddingLeft() + getRowPaddingRight();
        return (gridWidth - listPadding - mItemMargin * (mColumnCount + 1)) / mColumnCount;
    } 
```

[slide]
### 3.3.1 EtsyStaggered核心布局算法分析.
**DynamicHeightItemView自动确定试图尺寸**
```
 @Override
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
		if (mHeightRatio > 0.0) {
			// set the image views size
			int width = MeasureSpec.getSize(widthMeasureSpec);
			int height = (int) (width * mHeightRatio);
			setMeasuredDimension(width, height);
		}
		else {
			super.onMeasure(widthMeasureSpec, heightMeasureSpec);
		}
	} 
```

[slide]
### 3.3.1 EtsyStaggered核心布局算法分析.
**StaggeredGridView onMeasureChild**
```
    @Override
    protected void onMeasureChild(final View child, final LayoutParams layoutParams) {
        ...
        else {
            if (DBG) Log.d(TAG, "onMeasureChild BEFORE position:" + position +
                    " h:" + getMeasuredHeight());
            // measure it to the width of our column.
            int childWidthSpec = MeasureSpec.makeMeasureSpec(mColumnWidth, MeasureSpec.EXACTLY);
            int childHeightSpec;
            if (layoutParams.height > 0) {
                childHeightSpec = MeasureSpec.makeMeasureSpec(layoutParams.height, MeasureSpec.EXACTLY);
            }
            else {
                childHeightSpec = MeasureSpec.makeMeasureSpec(LayoutParams.WRAP_CONTENT, MeasureSpec.UNSPECIFIED);
            }
            child.measure(childWidthSpec, childHeightSpec);
        }

        final int childHeight = getChildHeight(child);
        setPositionHeightRatio(position, childHeight);

        if (DBG) Log.d(TAG, "onMeasureChild AFTER position:" + position +
                " h:" + childHeight);
    }
```

[slide]
### 3.3.1 EtsyStaggered核心布局算法分析.
**StaggeredGridView onLayoutChild**
```
    // NOTE : Views will either be layout out via onLayoutChild
    // OR
    // Views will be offset if they are active but offscreen so that we can recycle!
    // Both onLayoutChild() and onOffsetChild are called after we measure our view
    // see ExtensibleListView.setupChild();

    @Override
    protected void onLayoutChild(final View child,
                                 final int position,
                                 final boolean flowDown,
                                 final int childrenLeft, final int childTop,
                                 final int childRight, final int childBottom) {
        if (isHeaderOrFooter(position)) {
            layoutGridHeaderFooter(child, position, flowDown, childrenLeft, childTop, childRight, childBottom);
        }
        else {
            layoutGridChild(child, position, flowDown, childrenLeft, childRight);
        }
    }
```

[slide]
### 3.3.1 EtsyStaggered核心布局算法分析.
**StaggeredGridView layoutGridChild**
```
    private void layoutGridChild(final View child, final int position,
                                 final boolean flowDown,
                                 final int childrenLeft, final int childRight) {
        // stash the bottom and the top if it's higher positioned
        int column = getPositionColumn(position);
        ...
        if (flowDown) {
            gridChildTop = mColumnBottoms[column]; // the next items top is the last items bottom
            gridChildBottom = gridChildTop + (getChildHeight(child) + verticalMargins);
        }
        else {
            gridChildBottom = mColumnTops[column]; // the bottom of the next column up is our top
            gridChildTop = gridChildBottom - (getChildHeight(child) + verticalMargins);
        }

        // we also know the column of this view so let's stash it in the
        // view's layout params
        GridLayoutParams layoutParams = (GridLayoutParams) child.getLayoutParams();
        layoutParams.column = column;
        ...
        child.layout(childrenLeft, gridChildTop, childRight, gridChildBottom);
    }
```

[slide]
### 3.3.1 EtsyStaggered核心布局算法分析.
**StaggeredGridView onSizeChanged**
```
    @Override
    protected void onSizeChanged(int w, int h) {
    	super.onSizeChanged(w, h);
        boolean isLandscape = isLandscape();
        int newColumnCount = isLandscape ? mColumnCountLandscape : mColumnCountPortrait;
        if (mColumnCount != newColumnCount) {
            mColumnCount = newColumnCount;
            mColumnWidth = calculateColumnWidth(w);
            mColumnTops = new int[mColumnCount];
            mColumnBottoms = new int[mColumnCount];
            mColumnLefts = new int[mColumnCount];
            mDistanceToTop = 0;
            ...
            // if we have data
            if (getCount() > 0 && mPositionData.size() > 0) {
                onColumnSync();
            }

            requestLayout();
        }
    }

```

[slide]
### 3.3.1 EtsyStaggered核心布局算法分析.
**StaggeredGridView onColumnSync**
```
    /***
     * Our mColumnTops and mColumnBottoms need to be re-built up to the
     * mSyncPosition - the following layout request will then
     * layout the that position and then fillUp and fillDown appropriately.
     */
    private void onColumnSync() {
        ...
        mPositionData.clear();
        // rebuilding our GridItemRecord collection
        for (int pos = 0; pos < syncPosition; pos++) {
            ...
            final GridItemRecord rec = getOrCreateRecord(pos);
            ...
                // what's the next column down ?
                final int column = getHighestPositionedBottomColumn();
                ...
                rec.column = column;
            ...
        }
        ...
    }
```

[slide]
### 3.3.1 EtsyStaggered核心布局算法分析.
**StaggeredGridView getHighestPositionedBottomColumn**
```
    private int getHighestPositionedBottomColumn() {
        int columnFound = 0;
        int highestPositionedBottom = Integer.MAX_VALUE;
        // the highest positioned bottom is the one with the lowest value :D
        for (int i = 0; i < mColumnCount; i++) {
            int bottom = mColumnBottoms[i];
            if (bottom < highestPositionedBottom) {
                highestPositionedBottom = bottom;
                columnFound = i;
            }
        }
        return columnFound;
    }

```

[slide]
# 谢谢！