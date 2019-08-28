---
layout: post
title:  Android 在RecyclerView中实现置顶功能
author: shaDowZwy
---
在我上一篇博客中已经介绍了在ListView中实现置顶功能，后来也想在非常流行的RecyclerView中实现一下，所以就写下了这篇博客。

-----

如果是之前没有阅读过的，建议先去阅读下 [Android 在ListView中实现置顶功能](https://shadowzwy.github.io/Android-%E5%9C%A8ListView%E4%B8%AD%E5%AE%9E%E7%8E%B0%E7%BD%AE%E9%A1%B6%E5%8A%9F%E8%83%BD/) 。

效果如图哈
![](https://github.com/shaDowZwy/Douya/blob/master/device-2016-03-15-174713.mp4_1458035348.gif?raw=true)





**其实原理还是一样，就是adapter里的数据排序**
可是在RecyclerView中的adapter是需要继承于RecyclerView.adapter的，也就是说它没有提供给我们需要的ArrayAdapter的，当时我就方了，那岂不是要自己写？TnT

不过机智的我还是先谷歌了一下，果然是谷歌大法好啊，我发现了一个国外大牛已经写好了的ArrayAdapter for RecyclerView  [ArrayAdapter](https://gist.github.com/passsy/f8eecc97c37e3de46176)

我阅读了他的源码后觉得初步满足我的需求啊，然后果断就get了它！然后在它的基础上修改了一下代码，达到我的使用需求。代码如下：

```java

public abstract class RecyclerArrayAdapter<T, VH extends RecyclerView.ViewHolder>
        extends RecyclerView.Adapter<VH> {

    private List<T> mObjects;

    public RecyclerArrayAdapter(final List<T> objects) {
        mObjects = objects;
    }

    /**
     * Adds the specified object at the end of the array.
     *
     * @param object The object to add at the end of the array.
     */
    public void add(final T object) {
        mObjects.add(object);
        notifyItemInserted(getItemCount() - 1);
    }

    /**
     * Remove all elements from the list.
     */
    public void clear() {
        final int size = getItemCount();
        mObjects.clear();
        notifyItemRangeRemoved(0, size);
    }

    @Override
    public int getItemCount() {
        return mObjects.size();
    }

    public T getItem(final int position) {
        return mObjects.get(position);
    }

    public long getItemId(final int position) {
        return position;
    }

    /**
     * Returns the position of the specified item in the array.
     *
     * @param item The item to retrieve the position
     * @return The position of the specified item.
     */
    public int getPosition(final T item) {
        return mObjects.indexOf(item);
    }

    /**
     * Inserts the specified object at the specified index in the array.
     *
     * @param object The object to insert into the array.
     * @param index  The index at which the object must be inserted.
     */
    public void insert(final T object, int index) {
        mObjects.add(index, object);
        notifyItemInserted(index);

    }

    /**
     * Removes the specified object from the array.
     *
     * @param object The object to remove.
     */
    public void remove(T object) {
        final int position = getPosition(object);
        mObjects.remove(object);
        notifyItemRemoved(position);
    }

    /**
     * Sorts the content of this adapter using the specified comparator.
     *
     * @param comparator The comparator used to sort the objects contained in this adapter.
     */
    public void sort(Comparator<? super T> comparator) {
        Collections.sort(mObjects, comparator);
        notifyItemRangeChanged(0, getItemCount());
    }

    /**
     * 添加整个链表
     */
    public void addAll(List<T> list) {
        final int size = getItemCount();
        mObjects.addAll(list);
        notifyItemRangeChanged(0, size);
    }

}

```

从代码中可见，这是一个继承于RecyclerView.adapter的**泛型抽象类**，通过泛型T来绑定数据类型，在类中使用**mObjects**这个链表来维护数据的操作，并且在同时通知更新RecyclerView.adapter，如果不太了解这些通知方法的，可以去官方文档上看看，这里就不细说了哈。

而且可见我也就只加了个**addAll()**的方法 =_= ，因为根据上一篇博客所说的那样，需要这个方法来进行实时置顶交互。

OK！那这样就有了我们的ArrayAdapter了。那我们就要开始下一步了！
让我们使用这个已经封装好了的ArrayAdapter，代码如下：

```java

public class RecyclerViewAdapter extends RecyclerArrayAdapter<Session, RecyclerViewAdapter.RecyclerViewHolder> {


    private LayoutInflater mInflater;

    private ItemOnLongClickListener itemListener;


    public RecyclerViewAdapter(Context context) {
        super(new ArrayList<Session>());
        mInflater = LayoutInflater.from(context);
    }

    @Override
    public void onBindViewHolder(RecyclerViewHolder holder, final int position) {
        Session session = getItem(position);
        holder.textView.setText(String.valueOf(session.getTop()));
        holder.imageView.setImageResource(session.getAvatar());
        //自己实现itemClickListener
        holder.mItemView.setOnLongClickListener(new View.OnLongClickListener() {
            @Override
            public boolean onLongClick(View v) {
                //回调
                itemListener.itemLongClick(getItem(position));
                return false;
            }
        });
        if (session.getTop() == 1) {
            holder.mItemView.setBackgroundResource(R.drawable.bg_top_item_selector);
        } else {
            holder.mItemView.setBackgroundResource(R.drawable.bg_item_selector);
        }
    }

    @Override
    public RecyclerViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        //  使用这种解析方式 RecyclerView的match parent 属性才不会失效
        return new RecyclerViewHolder(mInflater.inflate(R.layout.itemview, parent, false));
    }


    public void setItemListener(ItemOnLongClickListener listener) {
        itemListener = listener;
    }

    public void updateData(List<Session> list) {
        clear();
        addAll(list);
    }

    public static class RecyclerViewHolder extends RecyclerView.ViewHolder {

        TextView textView;

        ImageView imageView;

        View mItemView;

        public RecyclerViewHolder(View itemView) {
            super(itemView);
            mItemView = itemView;
            textView = (TextView) itemView.findViewById(R.id.textView);
            imageView = (ImageView) itemView.findViewById(R.id.avatar_img);
        }
    }


    public interface ItemOnLongClickListener {

        void itemLongClick(Session session);
    }

}

```

如代码可见，我们继承了这个泛型抽象类ArrayAdapter，实现了RecyclerView需要的adapter，实现很简单，就是一个普通的adapter。

因为RecyclerView没有itemClick的监听器，所以这个要自己实现点击事件回调，相信使用过的应该都知道。还有，通过实现ArrayAdapter可以发现，RecyclerView确实是**解耦性**比较高，虽然代码量多一点，但是逻辑比较清晰，可控性也很好，值得深入使用。

那接下来所要实现的就是交互了，这跟我上一篇博客所说的差不多，只是因为是**RecyclerView**的不同，代码所处理的不同。

同样使用了Dialogfragment作为切换置顶的形式，连代码都不用改呢。。。

```java

public class PopupDialogFragment extends DialogFragment {

    private DialogItemOnClickListener itemOnClickListener;


    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {

        View view = inflater.inflate(R.layout.popview, null);
        TextView onTopTv = (TextView) view.findViewById(R.id.on_top_tv);
        TextView cancelTv = (TextView) view.findViewById(R.id.cancel_top_tv);

        Bundle bundle = getArguments();
        int isTop = bundle.getInt(MainActivity.TOP_STATES);
        if (isTop == 1) {
            onTopTv.setVisibility(View.GONE);
            cancelTv.setVisibility(View.VISIBLE);
        } else if (isTop == 0) {
            onTopTv.setVisibility(View.VISIBLE);
            cancelTv.setVisibility(View.GONE);
        }

        onTopTv.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                getDialog().dismiss();
                itemOnClickListener.onTop();
            }
        });

        cancelTv.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                getDialog().dismiss();
                itemOnClickListener.onCancel();
            }
        });


        getDialog().getWindow().requestFeature(STYLE_NO_TITLE);
        setStyle(STYLE_NO_FRAME, android.R.style.Theme_Light);
        setCancelable(true);
        getDialog().getWindow().setBackgroundDrawableResource(R.color.write_bg);
        return view;
    }


    public void setItemOnClickListener(DialogItemOnClickListener itemOnClickListener) {
        this.itemOnClickListener = itemOnClickListener;
    }


    public interface DialogItemOnClickListener {

        void onTop();

        void onCancel();

    }


}

```

接下来就是回调了，在MainActivity中实现回调，代码如下：
			
```java
mRecyclerViewAdapter.setItemListener(new RecyclerViewAdapter.ItemOnLongClickListener() {
            @Override
            public void itemLongClick(final Session session) {
                Bundle bundle = new Bundle();
                bundle.putInt(TOP_STATES, session.getTop());
                PopupDialogFragment popupDialog = new PopupDialogFragment();
                popupDialog.setArguments(bundle);
                popupDialog.setItemOnClickListener(new PopupDialogFragment.DialogItemOnClickListener() {
                    @Override
                    public void onTop() {
                        //置顶
                        session.setTop(1);
                        session.setTime(System.currentTimeMillis());
                        refreshView();
                    }

                    @Override
                    public void onCancel() {
                        //取消
                        session.setTop(0);
                        session.setTime(System.currentTimeMillis());
                        refreshView();
                    }
                });
                popupDialog.show(getFragmentManager(), "popup");
            }
});

```

最后还是一样不变的Session，实现置顶的数据排序，代码如下：

```java
public class Session implements Serializable, Comparable {

    /**
     * 是否置顶
     */
    public int top;

    /**
     * 置顶时间
     **/
    public long time;

    /**
     * 头像
     */
    public int avatar;


    public long getTime() {
        return time;
    }

    public void setTime(long time) {
        this.time = time;
    }

    public int getTop() {
        return top;
    }

    public void setTop(int top) {
        this.top = top;
    }

    public int getAvatar() {
        return avatar;
    }

    public void setAvatar(int avatar) {
        this.avatar = avatar;
    }

    @Override
    public int compareTo(Object another) {
        if (another == null || !(another instanceof Session)) {
            return -1;
        }

        Session otherSession = (Session) another;
        /**置顶判断 ArrayAdapter是按照升序从上到下排序的，就是默认的自然排序
         * 如果是相等的情况下返回0，包括都置顶或者都不置顶，返回0的情况下要
         * 再做判断，拿它们置顶时间进行判断
         * 如果是不相等的情况下，otherSession是置顶的，则当前session是非置顶的，应该在otherSession下面，所以返回1
         *  同样，session是置顶的，则当前otherSession是非置顶的，应该在otherSession上面，所以返回-1
         * */
        int result = 0 - (top - otherSession.getTop());
        if (result == 0) {
            result = 0 - compareToTime(time, otherSession.getTime());
        }
        return result;
    }

    /**
     * 根据时间对比
     * */
    public static int compareToTime(long lhs, long rhs) {
        Calendar cLhs = Calendar.getInstance();
        Calendar cRhs = Calendar.getInstance();
        cLhs.setTimeInMillis(lhs);
        cRhs.setTimeInMillis(rhs);
        return cLhs.compareTo(cRhs);
    }

}

```

以上便是所有了，点击这里有源码 [GitHub地址](https://github.com/shaDowZwy/AtTopRecyclerView)