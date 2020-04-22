---
title: 【Android】使用 PagerAdapter notifyDataSetChange 不刷新问题分析
date: 2020-04-22 22:50:10
categories: android
tags: [android]
---

## 1. 使用 PagerAdapter notifyDataSetChange 不刷新问题分析

无论是普通的 ViewPager 视图，还是用 Fragment，当我们刷新数据后调用 notifyDataSetChange 后，往往会发现当前界面并没有预想的触发刷新，根本原因在于 ViewPager 的缓存机制判定数据未发生变化，从而不触发刷新，及时数据确实发生了改变。

### 1.1 未触发刷新效果及分析
假设当前 page 是第一页，第一个数据发生变更，此时调用 notifyDataSetChange 后页面并没有变化，原因在于当前页面没有触发强制刷新，仅仅是从缓存中取数据而已。

#### 1.1.1 不刷新模拟效果
ViewPager 默认缓存数是1，即当前页+缓存，总共2，从效果图也可以看出更新数据后，当前页和滑动一页并不会销毁改页，在两页之后回到第一页，之前的页才被销毁，重新创建。

此页面有三个 view 在 ViewPager 中，当点击按钮会更新第一个 view 中的文字，在没有重写 getItemPosition 下，效果如下所示

![vp_2.gif](https://i.loli.net/2020/04/22/OgJVmaLwPqNjShb.gif)

#### 1.1.2 不刷新源码分析

当然，我们可以从 ViewPager 中看到，当我们调用 notifyDataSetChange 后会回调 VP 的 `void dataSetChanged() `，如下：

```
void dataSetChanged() {
	// This method only gets called if our observer is attached, so mAdapter is non-null.

    final int adapterCount = mAdapter.getCount();
    mExpectedAdapterCount = adapterCount;
    boolean needPopulate = mItems.size() < mOffscreenPageLimit * 2 + 1
            && mItems.size() < adapterCount;
    int newCurrItem = mCurItem;

    boolean isUpdating = false;
    for (int i = 0; i < mItems.size(); i++) {
        final ItemInfo ii = mItems.get(i);
		//========核心方法=========================
        final int newPos = mAdapter.getItemPosition(ii.object);

        if (newPos == PagerAdapter.POSITION_UNCHANGED) {
            continue;
        }

        if (newPos == PagerAdapter.POSITION_NONE) {
            mItems.remove(i);
            i--;

            if (!isUpdating) {
                mAdapter.startUpdate(this);
                isUpdating = true;
            }

            mAdapter.destroyItem(this, ii.position, ii.object);
            needPopulate = true;

            if (mCurItem == ii.position) {
                // Keep the current item in the valid range
                newCurrItem = Math.max(0, Math.min(mCurItem, adapterCount - 1));
                needPopulate = true;
            }
            continue;
        }
		//...省略一大堆
	}
	//...省略一大堆
}
```

首先会遍历所有缓存，通过 `mAdapter.getItemPosition(obj)` 判断是否需要销毁重建，该方法默认值 `POSITION_UNCHANGED` 为-1，即默认数据没有变化。所以我们只需要重写这个方法，确定当前位置的数据是否应该销毁，当然有很多 demo 都是直接建议销毁，即返回 POSITION_NONE。本身 ViewPager 设计并不是为频繁变化的数据，所以数据变化频繁或者为了性能更好，尽可能使用 RecyclerView + PagerSnapHelper 替换 VP。

#### 1.1.3 重写 adapter 中方法实现刷新 

好，为了继续使用 VP 来更新，应该动态的判断这个 `getItemPosition`，本身 VP 性能一般，一刀切不适合这种场景，所以我们通过 `setTag` 来解决这个数据关联问题，如下示例：
   
```
private List<HashMap<String, String>> data = new ArrayList<>();

class Adapter extends PagerAdapter {
	//...省略其他方法
	@NonNull
    @Override
    public Object instantiateItem(@NonNull ViewGroup container, int position) {
        TextView view = (TextView) LayoutInflater.from(ViewPagerActivity.this).inflate(R.layout.layout_simple_text, container, false);
        String text = data.get(position).get("name");
        view.setText(text);
        view.setTag(-1, position);
        view.setTag(-2, text);
        container.addView(view);
        return view;
    }

    @Override
    public int getItemPosition(@NonNull Object object) {
        if (getCount() == 0)
            return POSITION_UNCHANGED;
        View view = (View) object;
        int pos = (int) view.getTag(-1);
        String msg = (String) view.getTag(-2);
        if (pos >= getCount()) {
            return POSITION_NONE;
        }
        return msg.equals(data.get(pos).get("name")) ? POSITION_UNCHANGED : POSITION_NONE;
    }
}
```

更改之后效果图：

在判断之后，数据修改之后可以及时反馈到界面上，代价是需要数据源标记原始数据位置，但仅仅是一个4位的int

![vp_2.gif](https://i.loli.net/2020/04/22/OgJVmaLwPqNjShb.gif)

### 1.2 为什么还要使用 PagerAdapter？

ViewPager 在普通的 View 页面，在今天，使用空间其实很小了，但在 Fragment 组合页面，配合 FragmentPagerAdapter，不得不说非常方便，如果在存在需要动态增减 Fragment，使用 `getItemPosition（obj）` 来减少创建销毁还是比较合适，当然使用场景是固定几个 fragment （2-3个）配合`setOffscreenPageLimit`缓存可以一次创建足够，如果有一堆页面（超过设置的缓存数量），这个创建销毁过程的消耗还是客观存在，不容小觑。

### 1.3 FragmentPagerAdapter 改造，更适合 Fragment 增减

#### 1.3.1 FragmentPagerAdapter 创建分析

使用 `FragmentPagerAdapter` 有页面替换需要，除了重写 `getItemPosition()` 还需要重写 `getItemId(position)`。为什么？因为 `fragment` 存在于 `FragmentManager` 中，通过 `mFragmentManager.findFragmentByTag(name)` 来找到之前的 fragment，也可以理解为** fragment 的缓存**，具体源码代码如下：

```java
@Override
public Object instantiateItem(ViewGroup container, int position) {
    if (mCurTransaction == null) {
        mCurTransaction = mFragmentManager.beginTransaction();
    }

    final long itemId = getItemId(position);

    // Do we already have this fragment?
    String name = makeFragmentName(container.getId(), itemId);
    Fragment fragment = mFragmentManager.findFragmentByTag(name);
    if (fragment != null) {
        if (DEBUG) Log.v(TAG, "Attaching item #" + itemId + ": f=" + fragment);
        mCurTransaction.attach(fragment);
    } else {
        fragment = getItem(position);
        if (DEBUG) Log.v(TAG, "Adding item #" + itemId + ": f=" + fragment);
        mCurTransaction.add(container.getId(), fragment,
                makeFragmentName(container.getId(), itemId));
    }
    if (fragment != mCurrentPrimaryItem) {
        fragment.setMenuVisibility(false);
        fragment.setUserVisibleHint(false);
    }

    return fragment;
}
```

如果没有重写该方法，那么，在增减数据后，即使 `getItemPosition` 判定数据变化，再通过 `makeFragmentName(container.getId(), itemId)` （itemId 默认是position）获取的 tag 还是不变，那么重新拿到的 fragment 和原来位置的 fragment 一样，，所以必须通过重写 `getItemId` 来修改 tag，使这数据源中的 fragment 和这个 tag 形成唯一个关联关系，一般唯一性用hashCode就足够了。示例代码如下：

```java
private List<Fragment> data = new ArrayList<>();

class PageAdapter extends FragmentPagerAdapter {

    PageAdapter(FragmentManager fm) {
        super(fm);
    }

    @Override
    public Fragment getItem(int position) {
        SimpleFragment fragment = (SimpleFragment) data.get(position);
        fragment.setPosition(position);
        return fragment;
    }

    @Override
    public int getCount() {
        return data.size();
    }

    @Override
    public long getItemId(int position) {
        return data.get(position).hashCode();
    }

    @Override
    public int getItemPosition(@NonNull Object object) {
        if (getCount() == 0)
            return POSITION_UNCHANGED;

        int position = ((SimpleFragment) object).getPosition();
        if (position >= getCount()) {
            return POSITION_NONE;
        }

        return data.get(position).hashCode() == object.hashCode() ? POSITION_UNCHANGED : POSITION_NONE;
    }
}

```
**注：** SimpleFragment 只是继承 Fragment 增加一个 position 参数及相应方法。

### 1.3.4 FragmentPagerAdapter 修改前后效果图示

效果图：操作中的删除为 data.remove(1);//删除第二个数据

1.仅修改 getItemPosition

![vp_3.gif](https://i.loli.net/2020/04/22/bAvY6T1UgLHkFEh.gif)


2.修改 getItemPosition 和 getItemId

![vp_5.gif](https://i.loli.net/2020/04/22/3K61nlM5RjyPaQF.gif)


完。

