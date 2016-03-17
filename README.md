# ContactsDome
Android 使用ListView的A-Z字母排序功能实现联系人模块，主要学习了ListView的A-Z字母排序功能以及根据输入框的输入值改变来过滤搜索结果，如果输入框里面的值为空，更新为原来的列表，否则为过滤数据列表，包括汉字转成拼音的功能

先来看一下最终效果图：
![](http://img.blog.csdn.net/20160302163517914)

现在我们来看下项目结构图 
![](http://img.blog.csdn.net/20160302163730995)

其实很多都是和上一篇文章一样的，我在这里就不多说啦，不是很明白的地方，可以看一下上一篇文章 
我还是按照项目中类的顺序把代码贴出来吧 
1.ContactSortModel
```Java
package com.adan.contactsdome.view;

public class ContactSortModel {

    private String name;//显示的数据
    private String sortLetters;//显示数据拼音的首字母

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSortLetters() {
        return sortLetters;
    }

    public void setSortLetters(String sortLetters) {
        this.sortLetters = sortLetters;
    }
}
```
2.还是一样EditTextWithDel类就不贴上代码了[Android 带清除功能的输入框控件EditTextWithDel](http://blog.csdn.net/qq_20785431/article/details/50762834)

3.PinyinComparator
```Java
package com.adan.contactsdome.view;

import java.util.Comparator;

/**
 * 用来对ListView中的数据根据A-Z进行排序，前面两个if判断主要是将不是以汉字开头的数据放在后面
 */
public class PinyinComparator implements Comparator<ContactSortModel> {

    public int compare(ContactSortModel o1, ContactSortModel o2) {
        //这里主要是用来对ListView里面的数据根据ABCDEFG...来排序
        if (o1.getSortLetters().equals("@")
                || o2.getSortLetters().equals("#")) {
            return -1;
        } else if (o1.getSortLetters().equals("#")
                || o2.getSortLetters().equals("@")) {
            return 1;
        } else {
            return o1.getSortLetters().compareTo(o2.getSortLetters());
        }
    }
}
···
4.PinyinUtils类，就是第一点所讲的PinYin4j.jar用于将汉字转换为拼音啦，这里就不粘贴代码啦，[探索PinYin4j.jar将汉字转换为拼音的基本用法](http://blog.csdn.net/qq_20785431/article/details/50730342) 
5.SideBar类就是ListView右侧的字母索引View
```Java
package com.adan.contactsdome.view;

import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.Typeface;
import android.util.AttributeSet;
import android.view.MotionEvent;
import android.view.View;
import android.widget.TextView;

import com.adan.contactsdome.R;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

/**
 * ListView右侧的字母索引View
 */
public class SideBar extends View {

    public static String[] INDEX_STRING = {"A", "B", "C", "D", "E", "F", "G", "H", "I",
            "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T", "U", "V",
            "W", "X", "Y", "Z"};

    private OnTouchingLetterChangedListener onTouchingLetterChangedListener;
    private List<String> letterList;
    private int choose = -1;
    private Paint paint = new Paint();
    private TextView mTextDialog;

    public SideBar(Context context) {
        this(context, null);
    }

    public SideBar(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public SideBar(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        init();
    }

    private void init() {
        setBackgroundColor(Color.parseColor("#FFFFFF"));
        letterList = Arrays.asList(INDEX_STRING);
    }

    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        int height = getHeight();// 获取对应高度
        int width = getWidth();// 获取对应宽度
        int singleHeight = height / letterList.size();// 获取每一个字母的高度
        for (int i = 0; i < letterList.size(); i++) {
            paint.setColor(Color.parseColor("#606060"));
            paint.setTypeface(Typeface.DEFAULT_BOLD);
            paint.setAntiAlias(true);
            paint.setTextSize(20);
            // 选中的状态
            if (i == choose) {
                paint.setColor(Color.parseColor("#4F41FD"));
                paint.setFakeBoldText(true);
            }
            // x坐标等于中间-字符串宽度的一半.
            float xPos = width / 2 - paint.measureText(letterList.get(i)) / 2;
            float yPos = singleHeight * i + singleHeight / 2;
            canvas.drawText(letterList.get(i), xPos, yPos, paint);
            paint.reset();// 重置画笔
        }
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        final int action = event.getAction();
        final float y = event.getY();// 点击y坐标
        final int oldChoose = choose;
        final OnTouchingLetterChangedListener listener = onTouchingLetterChangedListener;
        final int c = (int) (y / getHeight() * letterList.size());// 点击y坐标所占总高度的比例*b数组的长度就等于点击b中的个数.

        switch (action) {
            case MotionEvent.ACTION_UP:
                setBackgroundColor(Color.parseColor("#FFFFFF"));
                choose = -1;
                invalidate();
                if (mTextDialog != null) {
                    mTextDialog.setVisibility(View.GONE);
                }
                break;
            default:
                setBackgroundResource(R.drawable.bg_sidebar);
                if (oldChoose != c) {
                    if (c >= 0 && c < letterList.size()) {
                        if (listener != null) {
                            listener.onTouchingLetterChanged(letterList.get(c));
                        }
                        if (mTextDialog != null) {
                            mTextDialog.setText(letterList.get(c));
                            mTextDialog.setVisibility(View.VISIBLE);
                        }
                        choose = c;
                        invalidate();
                    }
                }
                break;
        }
        return true;
    }

    public void setIndexText(ArrayList<String> indexStrings) {
        this.letterList = indexStrings;
        invalidate();
    }

    /**
     * 为SideBar设置显示当前按下的字母的TextView
     *
     * @param mTextDialog
     */
    public void setTextView(TextView mTextDialog) {
        this.mTextDialog = mTextDialog;
    }

    /**
     * 向外公开的方法
     *
     * @param onTouchingLetterChangedListener
     */
    public void setOnTouchingLetterChangedListener(
            OnTouchingLetterChangedListener onTouchingLetterChangedListener) {
        this.onTouchingLetterChangedListener = onTouchingLetterChangedListener;
    }

    /**
     * 接口
     */
    public interface OnTouchingLetterChangedListener {
        void onTouchingLetterChanged(String s);
    }
}
```

6.SortAdapter数据的适配器类，这里我们需要用到的就是SectionIndexer接口，它能够有效地帮助我们对分组进行控制。使用SectionIndexer接口需要实现三个方法：getSectionForPosition(int position)，getPositionForSection(int section)，getSections()，我们只需要自行实现前面两个方法： 
（一）getSectionForPosition(int position)是根据ListView的position来找出当前位置所在的分组 
（二）getPositionForSection(int section)就是根据首字母的Char值来获取在该ListView中第一次出现该首字母的位置,也就是当前分组所在的位置
```Java
package com.adan.contactsdome;

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.SectionIndexer;
import android.widget.TextView;

import com.adan.contactsdome.view.ContactSortModel;

import java.util.List;

public class SortAdapter extends BaseAdapter implements SectionIndexer {
    private List<ContactSortModel> list = null;
    private Context mContext;

    public SortAdapter(Context mContext, List<ContactSortModel> list) {
        this.mContext = mContext;
        this.list = list;
    }

    /**
     * 当ListView数据发生变化时,调用此方法来更新ListView
     *
     * @param list
     */
    public void updateListView(List<ContactSortModel> list) {
        this.list = list;
        notifyDataSetChanged();
    }

    public int getCount() {
        return this.list.size();
    }

    public Object getItem(int position) {
        return list.get(position);
    }

    public long getItemId(int position) {
        return position;
    }

    public View getView(final int position, View view, ViewGroup arg2) {
        ViewHolder viewHolder = null;
        final ContactSortModel mContent = list.get(position);
        if (view == null) {
            viewHolder = new ViewHolder();
            view = LayoutInflater.from(mContext).inflate(R.layout.item_contact, null);
            viewHolder.tvTitle = (TextView) view.findViewById(R.id.tv_city_name);
            view.setTag(viewHolder);
            viewHolder.tvLetter = (TextView) view.findViewById(R.id.tv_catagory);
        } else {
            viewHolder = (ViewHolder) view.getTag();
        }

        int section = getSectionForPosition(position);

        if (position == getPositionForSection(section)) {
            viewHolder.tvLetter.setVisibility(View.VISIBLE);
            viewHolder.tvLetter.setText(mContent.getSortLetters());
        } else {
            viewHolder.tvLetter.setVisibility(View.GONE);
        }

        viewHolder.tvTitle.setText(this.list.get(position).getName());

        return view;

    }


    final static class ViewHolder {
        TextView tvLetter;
        TextView tvTitle;
    }

    public int getSectionForPosition(int position) {
        return list.get(position).getSortLetters().charAt(0);
    }

    public int getPositionForSection(int section) {
        for (int i = 0; i < getCount(); i++) {
            String sortStr = list.get(i).getSortLetters();
            char firstChar = sortStr.toUpperCase().charAt(0);
            if (firstChar == section) {
                return i;
            }
        }
        return -1;
    }

    @Override
    public Object[] getSections() {
        return null;
    }
}
```

7.MainActivity 对EditTextWithDel设置addTextChangedListener监听，当输入框内容发生变化根据里面的值过滤ListView，里面的值为空显示原来的列表和给ListView添加表头等

```Java
package com.adan.contactsdome;

import android.app.Activity;
import android.os.Bundle;
import android.text.Editable;
import android.text.TextUtils;
import android.text.TextWatcher;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ListView;
import android.widget.TextView;
import android.widget.Toast;

import com.adan.contactsdome.view.ContactSortModel;
import com.adan.contactsdome.view.EditTextWithDel;
import com.adan.contactsdome.view.PinyinComparator;
import com.adan.contactsdome.view.PinyinUtils;
import com.adan.contactsdome.view.SideBar;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class MainActivity extends Activity {
    private ListView sortListView;
    private SideBar sideBar;
    private TextView dialog, mTvTitle;
    private SortAdapter adapter;
    private EditTextWithDel mEtSearchName;
    private List<ContactSortModel> SourceDateList;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initViews();
    }

    private void initViews() {
        mEtSearchName = (EditTextWithDel) findViewById(R.id.et_search);
        sideBar = (SideBar) findViewById(R.id.sidrbar);
        dialog = (TextView) findViewById(R.id.dialog);
        mTvTitle = (TextView) findViewById(R.id.tv_title);
        sortListView = (ListView) findViewById(R.id.lv_contact);
        initDatas();
        initEvents();
        setAdapter();
    }

    private void setAdapter() {
        SourceDateList = filledData(getResources().getStringArray(R.array.contacts));
        Collections.sort(SourceDateList, new PinyinComparator());
        adapter = new SortAdapter(this, SourceDateList);
        sortListView.setAdapter(adapter);
    }

    private void initEvents() {
        //设置右侧触摸监听
        sideBar.setOnTouchingLetterChangedListener(new SideBar.OnTouchingLetterChangedListener() {
            @Override
            public void onTouchingLetterChanged(String s) {
                //该字母首次出现的位置
                int position = adapter.getPositionForSection(s.charAt(0));
                if (position != -1) {
                    sortListView.setSelection(position + 1);
                }
            }
        });

        //ListView的点击事件
        sortListView.setOnItemClickListener(new AdapterView.OnItemClickListener() {

            @Override
            public void onItemClick(AdapterView<?> parent, View view,
                                    int position, long id) {
                mTvTitle.setText(((ContactSortModel) adapter.getItem(position - 1)).getName());
                Toast.makeText(getApplication(), ((ContactSortModel) adapter.getItem(position)).getName(), Toast.LENGTH_SHORT).show();
            }
        });

        //根据输入框输入值的改变来过滤搜索
        mEtSearchName.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {

            }

            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {
                //当输入框里面的值为空，更新为原来的列表，否则为过滤数据列表
                filterData(s.toString());
            }

            @Override
            public void afterTextChanged(Editable s) {

            }
        });
    }

    private void initDatas() {
        sideBar.setTextView(dialog);
    }

    /**
     * 根据输入框中的值来过滤数据并更新ListView
     *
     * @param filterStr
     */
    private void filterData(String filterStr) {
        List<ContactSortModel> mSortList = new ArrayList<>();
        if (TextUtils.isEmpty(filterStr)) {
            mSortList = SourceDateList;
        } else {
            mSortList.clear();
            for (ContactSortModel sortModel : SourceDateList) {
                String name = sortModel.getName();
                if (name.toUpperCase().indexOf(filterStr.toString().toUpperCase()) != -1 || PinyinUtils.getPingYin(name).toUpperCase().startsWith(filterStr.toString().toUpperCase())) {
                    mSortList.add(sortModel);
                }
            }
        }
        // 根据a-z进行排序
        Collections.sort(mSortList, new PinyinComparator());
        adapter.updateListView(mSortList);
    }

    private List<ContactSortModel> filledData(String[] date) {
        List<ContactSortModel> mSortList = new ArrayList<>();
        ArrayList<String> indexString = new ArrayList<>();

        for (int i = 0; i < date.length; i++) {
            ContactSortModel sortModel = new ContactSortModel();
            sortModel.setName(date[i]);
            String pinyin = PinyinUtils.getPingYin(date[i]);
            String sortString = pinyin.substring(0, 1).toUpperCase();
            if (sortString.matches("[A-Z]")) {
                sortModel.setSortLetters(sortString.toUpperCase());
                if (!indexString.contains(sortString)) {
                    indexString.add(sortString);
                }
            }
            mSortList.add(sortModel);
        }
        Collections.sort(indexString);
        sideBar.setIndexText(indexString);
        return mSortList;
    }
}
```
布局文件就不贴出来了，现在看来是不是比上一篇还简单了呢 


