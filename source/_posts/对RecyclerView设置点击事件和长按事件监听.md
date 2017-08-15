---
title: 对RecyclerView设置点击事件和长按事件监听
date: 2016-05-15 08:53
---




RecyclerView是Google官方推出的用来取代ListView的控件，他的灵活性和可替代性都远胜于ListView，还可以配合Material Design的一些其他控件组合出炫酷的效果，只是默认状态下是没有点击和长按事件的，所以当在需要点击事件和长按事件监听的话需要自己写。这里有两种方式可以实现。
<!-- more -->
## 第一种方式：
在Adapter中自己定义两个接口，分别用来实现点击和长按，然后在Adapter的onBindViewHolder方法中为holder.itemView设置点击和长按监听，最后在我们的Activity中进行回调。

Adapter中代码如下：


```java

public class RecyclerAdapter extends RecyclerView.Adapter<MyViewHolder> {

    private LayoutInflater layoutInflater;
    private List<String> mData;
    private Context context;

    //定义点击事件和长按事件的接口
    public interface OnItemClickLister {
        void onItemClick(View view, int position);
        void onItemLongClick(View view, int position);
    }

    private OnItemClickLister mOnItemClickLister;

    public void setOnItemClickListener(OnItemClickLister listener) {
        this.mOnItemClickLister = listener;
    }

    // TODO: 2016/4/7  
    private List<Integer> mHeight;
    public RecyclerAdapter(Context context, List<String> mData) {
        this.context = context;
        this.mData = mData;
        layoutInflater = LayoutInflater.from(context);

        // TODO: 2016/4/7
        mHeight = new ArrayList<>();
        for (int i = 0; i < mData.size(); i++) {
            mHeight.add((int) (200 + Math.random() * 400));
        }
    }

    @Override
    public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = layoutInflater.inflate(R.layout.recycler_item, parent, false);
        MyViewHolder viewHolder = new MyViewHolder(view);
        return viewHolder;
    }


    
    @Override
    public void onBindViewHolder(final MyViewHolder holder, final int position) {
        // TODO: 2016/4/7  
        ViewGroup.LayoutParams layoutParams = holder.itemView.getLayoutParams();
        layoutParams.height = mHeight.get(position);
        holder.itemView.setLayoutParams(layoutParams);
        holder.img.setImageResource(R.mipmap.b);
        holder.titletTxt.setText(mData.get(position));
        
        //对holder.item进行设置点击和长按事件
        if (mOnItemClickLister != null) {
            holder.itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    int layoutPosition = holder.getLayoutPosition();
                    mOnItemClickLister.onItemClick(holder.itemView, position);

                }
            });

            holder.itemView.setOnLongClickListener(new View.OnLongClickListener() {
                @Override
                public boolean onLongClick(View v) {
                    mOnItemClickLister.onItemLongClick(holder.itemView, position);
                    
                    //这里返回ture，阻断事件继续传递，防止长按事件和点击事件冲突
                    return true;
                }
            });
        }

    }

    @Override
    public int getItemCount() {
        return mData.size();
    }


    public void addData(int pos) {
        mData.add(pos, "加入一个");
        notifyItemInserted(pos);
    }

    public void deleteDate(int pos) {
        mData.remove(pos);
        notifyItemRemoved(pos);
    }
}


class MyViewHolder extends RecyclerView.ViewHolder {

    ImageView img;
    TextView titletTxt;

    public MyViewHolder(View itemView) {
        super(itemView);

        titletTxt = (TextView) itemView.findViewById(R.id.titletTxt);
        img = (ImageView) itemView.findViewById(R.id.img);
    }
}
```
Activity中代码：
```java

public class RecyclerViewTest extends AppCompatActivity {

    private RecyclerView recycler;

    private List<String> mData;

    private RecyclerAdapter recyclerAdapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_recycler_view_test);
        recycler = (RecyclerView) findViewById(R.id.recycler);
        initData();
        recyclerAdapter = new RecyclerAdapter(this, mData);

//        RecyclerView.LayoutManager layoutManager = new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false);

        recycler.setLayoutManager(new StaggeredGridLayoutManager(2, StaggeredGridLayoutManager.VERTICAL));
        recycler.setAdapter(recyclerAdapter);
        
        //对点击事件和长按事件进行回调
        recyclerAdapter.setOnItemClickListener(new RecyclerAdapter.OnItemClickLister() {
            @Override
            public void onItemClick(View view, int position) {
                Toast.makeText(RecyclerViewTest.this, "点击一次" + position, Toast.LENGTH_SHORT).show();
                startActivity(new Intent(RecyclerViewTest.this, MainActivity.class));
            }

            @Override
            public void onItemLongClick(View view, int position) {
                Toast.makeText(RecyclerViewTest.this, "长按一次" + position, Toast.LENGTH_SHORT).show();
            }
        });
    }

    private void initData() {
        mData = new ArrayList<String>();
        for (int i = 'a'; i < 'z'; i++) {
            mData.add((char) i + "");
        }
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {

        getMenuInflater().inflate(R.menu.main, menu);

        return true;

    }


    @Override
    public boolean onOptionsItemSelected(MenuItem item) {

        switch (item.getItemId()) {
            case R.id.action_list:
                recycler.setLayoutManager(new LinearLayoutManager(this));
                break;
            case R.id.action_grid:
                recycler.setLayoutManager(new GridLayoutManager(this, 3));

                break;
            case R.id.action_hor:
                recycler.setLayoutManager(new StaggeredGridLayoutManager(5, StaggeredGridLayoutManager.HORIZONTAL));
                break;
            case R.id.action_staggeerd:

                break;
            case R.id.addone:
                recyclerAdapter.addData(1);
                break;
            case R.id.remove:
                recyclerAdapter.deleteDate(1);
                break;
        }
        return super.onOptionsItemSelected(item);
    }
}

```
用了一下下，感觉良好，并没有发现什么bug，只是有一个问题还是让人很困扰：如果这样自定义点击和长按事件，那岂不是在我们写的每一个adapter都要反复写着相同的代码？

总而言之，需要进行解耦。于是，就想到了RecyclerViwe的addOnItemTouchListener监听手势。

就是接下来的第二种方式。

## 第二种方式：

自定义OnItemToucheListener
```java
/**
 * Created by Sunny   [晨]  on 2016/5/10   09:22.
 * 高效、简洁、强大且无注释
 */

public class RecyclerItemClickListener implements RecyclerView.OnItemTouchListener {
    private GestureDetector gestureDetector;
    private OnItemClickListener onItemClickListener;
    public interface OnItemClickListener {
        public void onItemClick(View view, int position);

        public void onItemLongClick(View view, int position);
    }

    public RecyclerItemClickListener(Context context, final RecyclerView recyclerView, OnItemClickListener mOnItemClickListener) {
        onItemClickListener = mOnItemClickListener;
        gestureDetector = new GestureDetector(context, new GestureDetector.SimpleOnGestureListener() {
            @Override
            public boolean onSingleTapUp(MotionEvent e) {
                return true;
            }

            @Override
            public void onLongPress(MotionEvent e) {
                View childViewUnder = recyclerView.findChildViewUnder(e.getX(), e.getY());
                if (childViewUnder != null && onItemClickListener != null) {
                    onItemClickListener.onItemLongClick(childViewUnder, recyclerView.getChildPosition(childViewUnder));
                }
            }
        });


    }

    @Override
    public boolean onInterceptTouchEvent(RecyclerView rv, MotionEvent e) {

        View childViewUnder = rv.findChildViewUnder(e.getX(), e.getY());
        if (childViewUnder != null && onItemClickListener != null && gestureDetector.onTouchEvent(e)) {
            onItemClickListener.onItemClick(childViewUnder, rv.getChildPosition(childViewUnder));
            return true;
        }
        return false;
    }

    @Override
    public void onTouchEvent(RecyclerView rv, MotionEvent e) {

    }

    @Override
    public void onRequestDisallowInterceptTouchEvent(boolean disallowIntercept) {

    }
}
```

actity中调用recycler的addOnItemTouchListener方法，回调RecyclerItemClickListener中的点击和长按事件。
```java
        recycler_view.addOnItemTouchListener(new RecyclerItemClickListener(this, recycler_view, new RecyclerItemClickListener.OnItemClickListener() {
            @Override
            public void onItemClick(View view, int position) {
                switch (position) {
            
                }
            }

            @Override
            public void onItemLongClick(View view, int position) {
            
            }
        }));

```
毫无疑问更推荐第二种方式，降低了代码的耦合度，省去了很多重复的代码。

一直都对google在5.0时代发布的material design的简洁灵动的风格有一种微微中毒的感觉，并认为这才是未来。

然而伴随material design而来的许多android新增控件，却似乎并没有做好在实际应用中的准备。新的事物带来新的体验的同时也会带来新的麻烦和bug，不过，这一切都是可以克服的，谁叫它那么酷呢:)

