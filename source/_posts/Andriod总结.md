---
title: Andriod总结
date: 2018-11-20 11:08:09
tags:
categories: andriod
---

###  andriod进程保活手段
```php
1:利用不同的app进程使用广播来进行相互唤醒

2:它是利用系统的漏洞来启动一个前台的Service进程，与普通的启动方式区别在于，它不会在系统通知栏处出现一个Notification，看起来就如同运行着一个后台Service进程一样。

3:调用系统api启动一个前台的Service进程，这样会在系统的通知栏生成一个Notification，用来让用户知道有这样一个app在运行着，哪怕当前的app退到了后台.比如网易云音乐.
```

### 1.android安装

> 所需软件
> [jdk](https://www.oracle.com/technetwork/java/javase/downloads/index.html) :已经集成了jre,jdk是java的开发工具包,jre是java运行环境
> 

#### 1.1 jdk安装
直接下载安装即可

### 2.教程
#### 2.1.1 输入框,文本框,输入框自动填充与按钮
- 输入框自动填充
```android
src/main/res/layout/content_my.xml

<!--动态匹配搜索内容,completionThreshold指定第几个字符开始匹配-->
    <AutoCompleteTextView
        android:id="@+id/autoCompleteTextView1"
        android:hint="请输入搜索关键词"
        android:completionThreshold="2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
        
src/main/java/[company]/[appname]/MyActivity.java中onCreate方法

    /**
     * 第一步:初始化控件
     * 第二步:需要一个适配器
     * 第三步:初始化数据源--这数据源去匹配文本框中输入的内容
     * 第四步:将adpter与当前AutoCompleteTextView绑定
     */
    acTextView = (AutoCompleteTextView) findViewById(R.id.autoCompleteTextView1);
    ArrayAdapter<String> adapter = new ArrayAdapter<String>(this,
            android.R.layout.simple_list_item_1, res);
    acTextView.setAdapter(adapter);

src/main/java/[company]/[appname]/MyActivity.java中类变量

    private AutoCompleteTextView acTextView;
    private String[] res = {"beijing1","beijing2",
                            "beijing3", "shanghai1","shanghai2"};
```
- 文本框,输入框与按钮
```android
src/main/res/layout/content_my.xml

    <TextView
        android:id="@+id/textView1"
        android:text="我爱你"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="28sp"
        android:textColor="#000000"
        />

    <EditText android:id="@+id/edit_message"
        android:hint="请输入你的姓名"
        android:text="abcd"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        ></EditText>
        
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/button_send"
        android:onClick="sendMessage"/>
        
src/main/java/[company]/[appname]/MyActivity.java

    public void sendMessage(View view) {
        Intent intent = new Intent(this, DisplayMessageActivity.class);
        EditText editText = (EditText) findViewById(R.id.edit_message);
        String message = editText.getText().toString();
        intent.putExtra(EXTRA_MESSAGE, message);
        startActivity(intent);
    }
    
```

- 图片按钮
```android
<ImageButton
        android:id="@+id/imageButton1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:src="@drawable/ic_action_search"/>
```

#### 2.1.2 图片和开关转换
- 图片展示
```android
    <ImageView
        android:layout_width="match_parent"
        android:background="#f0f0f0"
        android:layout_height="140dp"
        android:src="@mipmap/keke"
        />
```
- 开关转换控制
```android
<ToggleButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:checked="true"
        android:textOff="关闭"
        android:textOn="开启"
        />
```
- togglebutton开关转换与图片联动
```android
<ToggleButton
        android:id="@+id/toggleButton1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:checked="true"
        android:textOff="关闭"
        android:textOn="开启"
        />
    <ImageView
        android:id="@+id/imageView1"
        android:background="#2f2"
        android:layout_width="140dp"
        android:layout_height="140dp" />
        
//动作(实现类)
public class MyActivity extends AppCompatActivity implements CompoundButton.OnCheckedChangeListener {
    //定义成员变量
    private ToggleButton tb;
    private ImageView img;
    
    protected void onCreate(Bundle savedInstanceState) {
        //初始化控件
        tb = (ToggleButton) findViewById(R.id.toggleButton1);
        img f= (ImageView) findViewById(R.id.imageView1);
        
        tb.setOnCheckedChangeListener(this);
    }
    
    public void onCheckedChanged(CompoundButton compoundButton, boolean b) {
        tb.setChecked(b);
        img.setBackgroundResource(b ? R.drawable.ic_action_search : R.mipmap.keke);
    }
}
```