---
layout: post
title:  Android 在ListView中实现置顶功能
author: shaDowZwy
---

在项目中实现了ListView置顶的功能，所以自己打算写一下博客记录下来。

-----

**其实实现起来还是挺简单的，核心思想是改变其adapter里的数据排序。**

效果如图哈



![](http://upload-images.jianshu.io/upload_images/1632203-8061194cab3bbc83.gif?imageMogr2/auto-orient/strip)


*那么开始吧*

1 首先你要实现ListView吧，这个很简单，就不多说了，其次是要自己继承       Arrayadapter<T>,因为要用到数据的**排序**，所以使用Arrayadapter<T>绑定你的数据。

```java

 public SessionItemAdapter extends ArrayAdapter<Session> {    
 
    Context mContext;

    /**
     * 不建议使用这种方式 将数据与adapter进行绑定，如果要进行数据更新等操作
     * 因为数据引用是相同的情况，会同步影响数据的变更。例如使用clean（）方法消除数据
     * 不仅仅消除了adapter里面的数据，还会清除了相同内存地址的数据源
     */
    public SessionItemAdapter(Context context, List<Session> sessions) {
        super(context, 0, sessions);
        mContext = context;
    }

    public SessionItemAdapter(Context context) {
        super(context, 0, new ArrayList<Session>());
        mContext = context;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        if (convertView == null) {
            convertView = new ItemView(mContext);
        }
        ItemView itemView = (ItemView) convertView;
        itemView.setText(String.valueOf(getItem(position).getTop()));
        itemView.setAvatar(getItem(position).getAvatar());
        if (getItem(position).getTop() == 1) {
            itemView.setBackgroundResource(R.drawable.bg_top_item_selector);
        } else {
            itemView.setBackgroundResource(R.drawable.bg_item_selector);
        }

        return convertView;
    }

    public void updateData(List<Session> sessionList) {
        clear();
        addAll(sessionList);
    }

}

```
   这是普通的listview的adapter，只是要注意的是构造函数那里的注释，因为如果一开始将数据源绑定在ArrayAdapter,将会影响数据源的更新，所以使用第二种构造方法先传入空的list，在**updateData()**方法中实时更新数据，才能呈现出置顶效果。

2 我使用了Dialogfragment作为切换置顶的一种形式，里面使用了回调接口，我们在回调接口里改变数据，实现置顶。

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

可以从代码中看见数据里的top字段是实现置顶的判断，分别是1为置顶，0为不置顶。然后在下面代码里实现回调，设置数据改变。

```java
      mListView.setOnItemLongClickListener(new AdapterView.OnItemLongClickListener() {
            @Override
            public boolean onItemLongClick(AdapterView<?> parent, View view, final int position, long id) {
                final Session session = (Session) parent.getItemAtPosition(position);
                Bundle bundle = new Bundle();
                bundle.putInt(TOP_STATES, session.getTop());
                PopupDialogFragment popupDialog = new PopupDialogFragment();
                popupDialog.setArguments(bundle);
                popupDialog.setItemOnClickListener(new PopupDialogFragment.DialogItemOnClickListener() {
                    @Override
                    public void onTop() {
                        session.setTop(ON_TOP);
                        //设置置顶时间
                        session.setTime(System.currentTimeMillis());
                        refreshView();
                    }

                    @Override
                    public void onCancel() {
                        session.setTop(CANCEL_TOP);
                        refreshView();
                    }

                });
                ;
                popupDialog.show(getFragmentManager(), "POPUP");
                return true;
            }
        });
```

实时更新数据

```java
    private void refreshView() {
        //如果不调用sort方法，是不会进行排序的，也就不会调用compareTo
        Collections.sort(sessionList);
        itemAdapter.updateData(sessionList);
    }
```

3 重点应该是传入ArrayAdapter的数据**Session**，它跟普通的数据类型不会有太大的区别，只是让它实现了Comparable接口，重写了compareTo()方法，从而实现了置顶功能。

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
         * 如果是不相等的情况下，otherSession是置顶的，则当前session是非置顶的，
         * 应该在otherSession下面，所以返回1
         * 同样，session是置顶的，则当前otherSession是非置顶的，
         * 应该在otherSession上面，所以返回-1
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

相信注释已经写的很清楚了，通过1,0，-1的判断实现置顶。




这里是demo源码 [GitHub地址](https://github.com/shaDowZwy/AtTopListView)