## Fragment
Fragment允许将Activity拆分成多个独立封装的可重用组件，使得每个组件有了自己的生命周期和UI布局。Fragment有助于解决多设备适配问题，尤其是手机和平板电脑之间的适配。另外Fragment也展现出很好的动态构建UI的能力，允许在Activity的布局中动态添加、删除、更换Fragment。下面的图片展示了用Fragment提升手机和平板电脑之间布局适宜性的例子。

![](https://lh3.googleusercontent.com/-JkPDg8wIzNM/Vf9lWXteYxI/AAAAAAAAA94/eHXfjLevN78/s560-Ic42/184532_IjRZ_730588.png)

### 1. Fragment生命周期
先来看一张生命周期图
![](https://lh3.googleusercontent.com/-EC0wugOiZt0/Vf9_u_ta-jI/AAAAAAAAA-E/8QCih2yRC0g/s512-Ic42/184542_fq1V_730588.png)
这张Fragment生命周期图详细展示了Fragment各个状态之间的转变，这里给出一些解释和说明。  
首先要说明的是`onCreate()`和`onDestroy()`有可能被跳过，当在`onCreate()`方法中调用了`setRetainInstance(true)`以后，当Fragment重建的时候这两个方法就会被跳过，并且Fragment的默认构造函数也不会被调用。因此对于`Configuration change`(譬如屏幕旋转)后，希望Fragment以不变形式存在的，可以在`onCreate()`方法中调用`setRetainInstance(true)`。  
下面分别讲解几个主要的生命周期方法：
#### 1.1 onAttach() 和 onDetach()
`onAttach()`被调用时说明Fragment已经绑定到Activity，从这个方法开始包括以后都可以使用使用`getActivity()`获取绑定的Activity，也可在此方法中捕获Activity中实现的通信接口(见`4.3节`)。`onDetach()`调用在Activity和Fragment分离时候。

#### 1.2 onCreate()
此方法被调用时，Fragment的View层次尚未创建好，可以在这个方法中使用`getArgumnets()`获取Fragment初始化参数。如果有背景线程需要运行(I/O或者网络操作线程等，譬如使用`Loader`来加载数据)，可以在这个方法里开始让他们运行。

#### 1.3 onCreateView()
如果Fragment是包含UI的，那么应该在这里绘制UI(譬如调用`inflate()`方法)并返回View对象。
>**Warning:** 不要将Fragment的View绑定到`Parent Container`上，即应该将`inflate()`的`attachToRoot`绑定参数设置为`false`。
>```java
>View view = inflater.inflate(R.layout.details, container, false);
>```

#### 1.4 onActivityCreated()
此方法被调用时，Activity的View层次已经创建完成，这意味着其他需要绑定到Activity的Fragment都已经绑定好了。这时候就可以放心的引用Activity的UI部件或者其他Fragment的UI部件。

#### 1.5 onStart() 与 onStop()
这一对方法将随着Activity对应方法的调用而被调用。

#### 1.6 onResume() 与 onPause()
这一对方法将随着Activity对应方法的调用而被调用。和Activity的`onPause()`方法一样，任何在用户离开Fragment后需要持久化的东西应该在`onPause()`这个方法里去实现。

### 2. Fragment的重要方法
#### 2.1 创建方法和构造函数
通常我们应该使用`Factory Method`来进行Fragment的创建，如下所示
```java
public static SimpleFragment newInstance(int fragId) {
    Bundle bundle = new Bundle();
    bundle.putInt(FRAG_ID, fragId);

    SimpleFragment fragment = new SimpleFragment();
    fragment.setArguments(bundle);
    return fragment;
}
```
这样做的好处在于，创建出来的Fragment对象是携带fragId参数的成型的Fragment。  
值得注意的是，获取Fragment的初始化参数应该在`onCreate()`而不能简单在`newInstance()`方法中将需要设置的初始化参数直接赋值给Fragment类的某个成员变量。这是由于当Fragment重新创建的时候，`newInstance()`并不会被调用，Android系统只会调用默认构造函数来创建Fragment，并在`onCreate(Bundle myBundle)`方法中将之前的初始化函数(Bundle数据)传入其中。

#### 2.2 其他重要方法
```java
View getView(); //获取Fragment的View，即onCreateView()方法返回的View
```

### 3. Fragment类型
#### 3.1 静态Fragment
在Activity的布局中，可以将某一个独立UI区域作为一个整体，在xml文件中声明为`<fragment>`标签，并且这样的标签必须指定与某一个特定的Fragment class名相关联。从而在Activity加载过程中，将`<fragment>`标签处替换为关联的那个Fragment class创建的Fragment UI并显示。这样的Fragment就是静态的Fragment。  
静态Fragment的好处在于编程简单，可以在横竖屏适配、多设备适配方面起到很好作用。缺点在于不能对Fragment区域的UI进行进行动态替换。以下给出一个简单的例子。
```xml
<!--fragment_my.xml-->
<!--xml file associated to MyFragment class-->

<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.example.wilson.sfragmentdemo.MyFragment">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:text="Blank Fragment" />

</RelativeLayout>
```

```
<!--activity_main.xml-->
<!--xml file associated to MainActivity-->

<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
    android:layout_height="match_parent" android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:paddingBottom="@dimen/activity_vertical_margin" tools:context=".MainActivity">

    <TextView
        android:id="@+id/textView"
        android:text="Activity Header"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <fragment
        class = "com.example.wilson.sfragmentdemo.MyFragment"
        android:id="@+id/myfrag"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/textView"/>

</RelativeLayout>
```
注意到上面的Activity xml文件中的`<fragment>`标签，需要指定`class`属性关联一个Fragment的class。  
接下来看看一个Fragment类是如何实现的
```java
public class MyFragment extends Fragment {

    private static final String FRAG_ID = "frag_id";

    private int mFragId;

    public static MyFragment newInstance(int FragId) {
        MyFragment fragment = new MyFragment();
        Bundle args = new Bundle();
        args.putInt(FRAG_ID, FragId);
        fragment.setArguments(args);
        return fragment;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //get fragment initialization parameters here
        if (getArguments() != null) {
            mFragId = getArguments().getInt(FRAG_ID);
        }
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_my, container, false);
    }

}
```
对照`第1节`、`第2节`的讲述，这里的`newInstance()`,`onCreate()`,`onCreateView()`这几个方法的意思就非常明白了。  
至于Activity类就只是一个普通的Activity类而已，没有什么需要特别讲述的内容。运行结果如下所示：

![](https://lh3.googleusercontent.com/-nP6C7m5BDW8/VgC3E1rRcrI/AAAAAAAAA-M/3hipzjoHr-g/s512-Ic42/device-2015-09-21-225208.png)

#### 3.2 动态Fragment
动态Fragment并不像静态Fragment那样Activity的xml布局文件的`<fragment>`标签以`class`属性来进行静态的声明和绑定，而是在Activity的UI加载完成以后可以对Fragment进行动态的添加、替换和删除。这样的动态特性带来了极大的灵活性，在不需要Activity销毁和重建的情况下就可以使Activity的全部或部分UI组件得到替换。  
这样的动态特性需要`FragmentManager`类来支持，对Fragment进行动态添加、替换或删除的过程是事务性的操作。因此基本上遵循下面的模式。
```java
FragmentManager fm = getSupportFragmentManager();
Fragment fragment = fm.findFragmentById(getFragmentContainerId());
if (fragment == null) { //fragment not exist
    fragment = createFragment();
    fm.beginTransaction() //返回FragmentTransaction对象
            .add(getFragmentContainerId(), fragment) //添加、替换和删除Fragment
            .commit();
}
```
先获取`FragmentManager`，再通过`FragmentManager`来启动事务性的添加、替换或删除Fragment的过程，最后务必需要调用`commit()`来提交更改。  
#####添加、替换或删除Fragment
```java
fragmentTransaction.add(R.id.fragment_container, fragment); //添加

fragmentTransaction.remove(fragment); //删除

fragmentTransaction.replace(R.id.fragment_container, fragment); //替换
```
替换相当于先删除已有的Fragment，在添加新的Fragment。`add`和`replace`方法都是接受两个参数，第一个参数是Activity布局文件中的`parent container`，这只需要是一个`ViewGroup`即可，譬如任何一种`Layout`都可以。Fragment的UI将作为这个`parent container`的`children`被动态挂载到Activity的UI层次中。

##### 添加Fragment进back stack
```java
fragmentTransaction.addToBackStack("fragementTag");
```
将fragment添入back stack以后，就可以在按Back键以后回到上一个Fragment的状态，行为和Activity的back stack类似。

#### 3.3. 单Activity单Fragment范例
通常Activity的UI是在Activity的xml布局文件中给出，并使用`setContentView()`来加载这个布局文件。但是事实上也可以将所有的UI都代理给Fragment来管理，Activity仅仅负责容纳一个`container`并加载这个Fragment到Container。这样做的好处是比仅仅使用Activity要灵活。  
先给出Main Activity的xml布局文件
```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
                xmlns:tools="http://schemas.android.com/tools"
                tools:context=".MainActivity"
                android:layout_height="fill_parent"
                android:layout_width="fill_parent"
                android:id="@+id/fragment_container">

</FrameLayout>
```
事实上就仅仅是一个`container`而已，接着来看看Fragment的xml布局文件。
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent" android:layout_height="match_parent">

    <TextView
        android:id="@+id/textView"
        android:text="@string/hello_world"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
</RelativeLayout>
```
作为实例，这里仅仅存在一个TextView，事实上你可以将这个UI扩展得很复杂。  
接下来看一下Fragment类
```java
public class SimpleFragment extends Fragment {
    public static final String FRAG_ID = "FRAG_ID";

    public static SimpleFragment newInstance(int fragId) {
        Bundle bundle = new Bundle();
        bundle.putInt(FRAG_ID, fragId);

        SimpleFragment fragment = new SimpleFragment();
        fragment.setArguments(bundle);
        return fragment;
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        if (container == null) {
            return null;
        }
        View view = inflater.inflate(R.layout.fragment_main, null);
        return view;
    }
}
```
和`3.1节`中的Fragment类没有什么差别，还是在`onCreateView()`来`inflate`我们自己定义的Fragment UI布局。  
再接着就应该是在Activity中来创建(`第2节`)并加载(`3.2节`)Fragment了，为了使得这里说的范例更加通用，先定义了一个抽象的`SingleFragmentActivity`，这个类的`onCreate()`方法将做我们的创建和加载Fragment的工作。但是这个类将创建的具体工作留给了子类去完成，另外继承这个抽象类的子类还需要给出获取`Container ID`和`Activity ID`的实现，换言之，继承这个抽象类的具体的Activity类需要实现以下三个方法：
```java
protected abstract int getActivityLayoutId();
protected abstract int getFragmentContainerId();
protected abstract Fragment createFragment();
```
这里先给出`SingleFragmentActivity`类的具体代码
```java
public abstract class SingleFragmentActivity extends AppCompatActivity {
    protected abstract int getActivityLayoutId();
    protected abstract int getFragmentContainerId();
    protected abstract Fragment createFragment();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(getActivityLayoutId());

        android.support.v4.app.FragmentManager fm = getSupportFragmentManager();
        Fragment fragment = fm.findFragmentById(getFragmentContainerId());
        if (fragment == null) { //fragment not exist
            fragment = createFragment();
            fm.beginTransaction() //add fragment to Activity
                    .add(getFragmentContainerId(), fragment)
                    .commit();
        }

    }
}
```
最后我们只需要去继承`SingleFragmentActivity`抽象类即可
```java
public class MainActivity extends SingleFragmentActivity {

    @Override
    protected int getActivityLayoutId() {
        return R.layout.activity_main;
    }

    @Override
    protected int getFragmentContainerId() {
        return R.id.fragment_container;
    }

    @Override
    protected Fragment createFragment() {
        return SimpleFragment.newInstance(-1);
    }
}
```
这里的Fragment创建工作就调用`Fragment`类的`newInstance()`来完成即可。

### 4. Fragment通信
#### 4.1 Activity向Fragment通信
这个较为简单，在`2.1节`已经给出，即Activity可以创建`Bunlde`数据包，然后调用`setArguments()`方法将Bundle数据包传递给Fragment。
```java
public static SimpleFragment newInstance(int fragId) {
    Bundle bundle = new Bundle();
    bundle.putInt(FRAG_ID, fragId);

    SimpleFragment fragment = new SimpleFragment();
    fragment.setArguments(bundle);
    return fragment;
}
```
另外可以使用`getActivity()`来获取Fragment所绑定的Activity。

#### 4.2 Fragment向Activity通信
推荐的方法是定义一个callBack接口并在Activity中做实现，在Fragment的`onAttach()`方法中捕获这个callBack接口的实现，接下来的Fragment生命周期中就都可以调用这个callBack接口。  
下面这个例子将展示这样的一个`Demo`：在Fragment UI中包含一个Button控件，点击这个Button控件会触发`FragmentButtonClickListener`接口中的`onFragmentButtonClickListener()`接口方法的调用，而该方法是在Activity中实现一个简单的打印日志，从而使得点击Fragment的Button会使得Activity打印一行日志。  
Activity的Layout与`3.3节`中完全一样，此处不再列举，Fragment的Layout仅仅增加了一个`Button`控件。
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent" android:layout_height="match_parent">

    <TextView
        android:id="@+id/textView"
        android:text="@string/hello_world"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

    <Button
        android:id="@+id/button"
        android:text="@string/hello_world"
        android:layout_below="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
</RelativeLayout>
```
接着看下Fragment类的变化
```java
public class SimpleFragment extends Fragment {
    public static final String FRAG_ID = "FRAG_ID";

    private FragmentButtonClickListener listener = null;

    public static SimpleFragment newInstance(int fragId) {
        Bundle bundle = new Bundle();
        bundle.putInt(FRAG_ID, fragId);

        SimpleFragment fragment = new SimpleFragment();
        fragment.setArguments(bundle);
        return fragment;
    }

    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        listener = (FragmentButtonClickListener) context;
        //concrete listener realisation is bind to listener
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        if (container == null) {
            return null;
        }
        View view = inflater.inflate(R.layout.fragment_main, null);
        Button button = (Button) view.findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                //call listener method here
                if (listener != null) {
                    listener.onFragmentButtonClickListener();
                }
            }
        });

        return view;
    }

    public interface FragmentButtonClickListener {
        public void onFragmentButtonClickListener();
    }
}
```
Fragment类多出了一个`FragmentButtonClickListener`接口并定义了`onFragmentButtonClickListener()`方法。在Fragment的`onAttach(Context context)`方法中，将试图捕获Activity中实现的接口。在`onCreateView()`方法中当点击`Button`时将触发使用`listener`来调用`onFragmentButtonClickListener()`。注意到此时`listener`已经绑定到具体实现了的接口，因此将调用子类(即Activity)中实现的那些接口方法的版本。是的，**这就是OOP**!
下面是Activity中的代码
```java
public class MainActivity extends SingleFragmentActivity
	implements SimpleFragment.FragmentButtonClickListener {

    @Override
    protected int getActivityLayoutId() {
        return R.layout.activity_main;
    }

    @Override
    protected int getFragmentContainerId() {
        return R.id.fragment_container;
    }

    @Override
    protected Fragment createFragment() {
        return SimpleFragment.newInstance(-1);
    }

    @Override
    public void onFragmentButtonClickListener() {
        Log.d("gyw", "Fragment Button is clicked");
    }
}
```
Activity代码与`3.3节`的唯一区别就在于实现了`SimpleFragment.FragmentButtonClickListener`这个接口。

#### 4.3 Fragment与Fragment之间的通信
一般不建议Fragment进行直接的通信，即Fragment之间可以通过绑定的Activity作为信道进行通信。当然事实上直接的通信也是可以的，在Fragment已经确知绑定到Acitivity上的其他Fragment可以调用`FragmentManager`的`findFragmentById()`方法或者`findFragmentByTag()`方法来直接获得其他Fragment的引用来进行通信。  
另外还有一种通信机制，就是在Fragment(CallerFragment)启动别的Fragment(CalleeFragment)时，可以调用`setTargetFragment()`方法来将自己设置为TargetFragment表面调用者身份，从而让跳转到CalleeFragment时，CalleeFragment可以通过调用`getTargetFragment()`得知是哪一个Fragment调用自己的。
```java
//In caller Fragment class body
CalleeFragment calleeFragment = new CalleeFragment();
calleeFragment.setTargetFragment(this, 0);
fragmentManager.beginTransaction()
	.add(calleeFragment, "work")
	.commit();
```

```java
//In callee Fragment class body
TextView textview = (TextView) getTargetFragment().getView().findViewById(R.id.textView);
```
更加完整具体的例子见`第5节`。

>**Tip:** 事实上Fragment与Activity或者Fragment之间的通信还可以使用更简便、更加低耦合的方式。Square的Otto库提供了简单利用事件总线来在不同的Android部件之间传递数据的解决方案。  
>https://github.com/square/otto

### 5. RetainedFragment
通常Fragment是存在UI界面的，加载Fragment意味着加载了Fragment的UI界面；然而也存在没有界面的Fragment，或者称作`RetainedFragment`，这样的Fragment也很重要，通常会被用在存在Background thread任务的情形。
在[Android多线程](https://github.com/byrcoder-docs/android_multithreading)文章的14.1节中提到

>**Reference:** 需要指出的是给出的Demo例子当遇到Configuration Change的时候(譬如屏幕旋转)是有bug的，潜在的bug包括onCreate()会重新被调用从而使得第一次的任务没有完成或者取消的情况下，又执行一次同样的任务。另外，更大的问题在于MyAsyncTask的中的mTextView将仍然指向第一次创建的Activity，这使得刷新UI存在潜在问题，另外也造成了第一次创建的Activity内存泄露了。这些问题的解决并非本文的重点，也超出本文的范畴，这里不再赘述。

解决的办法有多种，使用`RetainedFragment`就是一种好办法。下面通过一个简单的例子来解释。xml布局文件较为简单就不累述。`demo`使用一个UiFragment来作为界面，另一个`RetainedFragment`来执行和维护后台任务。后台任务执行会将进度刷新到UiFragment的`TextView`上。当旋转屏幕时，终止任务执行，并从头执行任务。先来看一下UiFragment的代码
```java
public class UiFragment extends Fragment {

    RetainedFragment mWorkFragment;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_main, container, false);

        Button button = (Button) view.findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                mWorkFragment.reStart(); //restart Button
            }
        });

        return view;
    }

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        FragmentManager fm = getFragmentManager();

        mWorkFragment = (RetainedFragment) fm.findFragmentByTag("work");
        
        if (mWorkFragment == null) {
            mWorkFragment = new RetainedFragment();
            mWorkFragment.setTargetFragment(this, 0);
            fm.beginTransaction()
                    .add(mWorkFragment, "work")
                    .commit();
        }
    }
}
```
`onCreateView`中为`Button`设置了一个监听器，这样使得可以利用这个`Button`来重新运行后台任务(未旋转屏幕或者旋转屏幕都是有效的)。注意到在`onActivityCreated`方法中，加载`RetainedFragment`从而新建后台线程。因为`mWorkFragment`并没有界面，因此并不需要挂载到某一个`container`下，可以使用`add(mWorkFragment, "TAG")`的方式进行加载启动。  
接下来看看`RetainedFragment`的代码，一段一段来讲。
```java
private static final int PUBLISH_PROGRESS = 0x001;
int mPosition = 0;
boolean mReady = false;
boolean mQuiting = false;

final Thread mThread = new Thread() {
    @Override
    public void run() {
        int max = 10000;
        
        while (true) {
            synchronized (this) {
                while (!mReady || mPosition > max-1) {
                    //not ready or work is completed
                    if (mQuiting) {
                        return;
                    }
                    try {
                        wait(); //wait until is ready
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                
                mPosition++;
                Message msg = mSimpleHandler.obtainMessage(PUBLISH_PROGRESS, mPosition, 0);
                mSimpleHandler.sendMessage(msg);
            }
        }
    }
};
```
`mThread`正是真正做工作的地方，是在另一个背景线程中执行工作。因为是`demo`因此仅仅是执行`mPosition++`这样简单的操作。当`!mReady || mPosition > max-1`条件满足时，即没有准备好或者工作已完成的时候，线程将阻塞等待，如果被取消(`mQuiting == true`)则线程直接终止返回。值得注意的是，这里并没有在线程中直接将`mPosition`设置到UI的`TextView`上，而是通过`Handler`发消息的方式，正是由于**Android不允许在非UI线程进行UI操作**。

```java
private SimpleHandler mSimpleHandler;

private static class SimpleHandler extends Handler {
    
    TextView mTextView = null;
    int mPosition = 0;
    
    public SimpleHandler(TextView textView) {
        mTextView = textView;
    }
    
    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
        switch (msg.what) {
            case PUBLISH_PROGRESS:
            mPosition = msg.arg1;
            Log.d("gyw", "pos = " + mPosition);
            mTextView.setText("pos = " + mPosition);
            break;
        }
    }
}
```
`Handler`处于UI线程中，因此可以在收到消息后安全地将`mPosition`的值设置到`mTextView`上。之所以使用`RetainFragment`可以避免本节一开始提到那些问题是因为`RetainFragment`会始终绑定到最新的Activity上，这使得我们有机会可以一直获得最新的Activity或者其他绑定到Activity的Fragment的UI元素。并且我们并不销毁Fragment，使得那些想保持的变量依然可以得到保持，譬如可以保持线程一直存在。

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setRetainInstance(true);
    mThread.start(); //begin background thread work
}
```
`setRetainInstance(true)`保证了`RetainFragment`不被销毁和重新创建，因此`mThread.start()`方法将仅仅执行一次，这正是我们所需要的。

```java
//we know that UI fragment have already bind now
@Override
public void onActivityCreated(Bundle savedInstanceState) {
    super.onActivityCreated(savedInstanceState);
    
    TextView textView = (TextView) getTargetFragment().getView()
            .findViewById(R.id.textView);
    mSimpleHandler = new SimpleHandler(textView);

    synchronized (mThread) {
        mReady = true;
        mThread.notify(); //pair to {mThread}.wait() in thread body
    }
}
```
当`onActivityCreated`方法被调用的时候，我们就可以确认其他的Fragment也已经绑定到Activity上，譬如我们的UiFragment，这样我们可以放心的获取`textView`，并将这个`textView`传递给`mSimpleHandler`。在这一切做完以后，`mThread`事实上还在阻塞等待我们信号。此时将`mReady`设置为`true`并唤醒后台线程。即使是屏幕旋转以后，该方法也会得到执行。

```java
    @Override
    public void onDetach() {
        super.onDetach();
        mPosition = 0;
        mReady = false;
    }
```
当遇到`Configuration Change`譬如屏幕旋转时，`onDetach()`会执行，通过`mReady = false`和`mPosition = 0`达到终止线程运行(其实是线程处于wait状态)并在屏幕旋转以后重新来过的效果。

```java
@Override
public void onDestroy() {
    super.onDestroy();
    synchronized (mThread) {
        mReady = false;
        mQuiting = true;
        mThread.notify();
    }
}
```
当遇到`Configuration Change`譬如屏幕旋转时，`onDestroy()`并不会被调用，只有当Fragment被清理的时候才会被调用。因此设置`mQuiting = true`，这事实上将使得线程返回并彻底终止。

```java
public void reStart() {
    synchronized (mThread) {
        mPosition = 0;
        mThread.notify();
    }
}
```
`UiFragment`的`Button`通过点击可以触发`reStart()`重新执行任务。

>**Tip:** 再次说明，当在`onCreate()`方法中设置`setRetainInstance(true)`将会使得在遇到`Configuration Change`譬如屏幕旋转时，Fragment的生命周期方法`onCreate()`和`onDestory()`被跳过。
