### Philm.MVP 实践——UI篇



#### 前言

philm感觉是一个经典的mvp项目，值得好好学一下。学习最好的方法就是实践。邯郸学步、西施效颦、是突破0的关键。



#### 1.基本流程

在philm项目中，UI交互总的来说就是View和Controller之间的交互，平时我们最常规的做法是在View中完成所有UI交互(切换)。

![这里写图片描述](http://img.blog.csdn.net/20151104112440553)



##### 1.1 Controller



``BaseController`` :  持有Display、控制Controller状态。

mInited = true:

状态变化是由BaseActivity.onResume() => MainController.init() => MainController.onInited() =>AllXXController.init()；

状态变化的结果：MainController.mInited = true=> AllXXController.mInited = true;

mInited = fasle:

状态变化是由BaseActivity.onPause() =>MainController.suspend() =>MainController.onSuspended()

=>AllXXController.suspend()；

状态变化的结果：MainController.mInited = false=> AllXXController.mInited = false;



``` java
public class BaseController {
    private Display display;
    private boolean mInited;

    public final void init() {
        mInited = true;
        onInited();
    }

    public final void suspend() {
        onSuspended();
        mInited = false;
    }

    protected void onInited() {
    }

    protected void onSuspended() {
    }


    public Display getDisplay() {
        return display;
    }

    public void setDisplay(Display display) {
        this.display = display;
    }

    public boolean ismInited() {
        return mInited;
    }
}
```



``BaseUiController``：

泛型``UC``：作为View中操作的CallBack，在View的OnResume()中调用attachUi()进行UI绑定；同时绑定UC;

``Ui``接口： Controller中定义的接口，View需要实现该接口；View在接口的实现中，可以定制满足View本身的需求。



``` java
public abstract class BaseUiController <U extends BaseUiController.Ui<UC>, UC>
        extends BaseController {

    public interface Ui<UC> {
        void setCallbacks(UC callbacks);

        boolean isModal();
    }

    public interface SubUi {
    }

    private final Set<U> mUis;
    private final Set<U> mUnmodifiableUis;

    public BaseUiController() {
        mUis = new CopyOnWriteArraySet<>();
        mUnmodifiableUis = Collections.unmodifiableSet(mUis);
    }

    protected final Set<U> getUis() {
        return mUnmodifiableUis;
    }

    protected synchronized final void populateUis() {
        for (U ui : mUis) {
            populateUi(ui);
        }
    }

    protected void onInited() {
        if (!mUis.isEmpty()) {
            for (U ui : mUis) {
                onUiAttached(ui);
                populateUi(ui);
            }
        }
    } 

    public synchronized final void attachUi(U ui) {
        Preconditions.checkArgument(ui != null, "ui cannot be null");
        Preconditions.checkState(!mUis.contains(ui), "UI is already attached");
        mUis.add(ui);
        ui.setCallbacks(createUiCallbacks(ui));

        if (ismInited()) {
            if (!ui.isModal() && !(ui instanceof SubUi)) {
                updateDisplayTitle(getUiTitle(ui));
            }
            onUiAttached(ui);
            populateUi(ui);
        }
    }

    protected void populateUi(U ui) {
    }

    protected void onUiAttached(U ui) {
    }

    protected String getUiTitle(U ui) {
        return null;
    }

    protected final void updateDisplayTitle(String title) {
        Display display = getDisplay();
        if (display != null) {
            display.setActionBarTitle(title);
        }
    }


    protected abstract UC createUiCallbacks(U ui);

    public synchronized final void detachUi(U ui) {
        Preconditions.checkArgument(ui != null, "ui cannot be null");
        Preconditions.checkState(mUis.contains(ui), "ui is not attached");
        onUiDetached(ui);
        ui.setCallbacks(null);

        mUis.remove(ui);
    }
    protected void onUiDetached(U ui) {
    }


}
```



``MainController``:

1. 持有其他所有XXController并初始化其他Controller；
   
2. 绑定``Display``,BaseActivity.onResume() => MainController.attachDisplay(mDisplay) => BaseController.setDisplay(mDisplay);
   
3. 取消绑定``Display`` ,BaseActivity.onPause() => MainController.detachDisplay(mDisplay) => BaseController.setDisplay(null);
   
4. 实现``MainControllerUiCallbacks ``并绑定到View，绑定流程：XX_View.onResume() => BaseUiController.attachUi(this) => XX_Controller.createUiCallbacks() => XX_View.setCallbacks();
   
   此过程使View持有了UiCallBacks，该View可以触发的事件通过UiCallBack传递到Controller中，Controller再进行分发；



``` java
@Singleton
public class MainController extends BaseUiController<MainController.MainControllerUi,
        MainController.MainControllerUiCallbacks> {

    private final ApplicationState mState;
    private final UserController mUserController; 

    @Inject
    public MainController(
            ApplicationState state, 
            UserController userController ) {
        super();
        mUserController = Preconditions.checkNotNull(userController, "UserController cannot be null"); 
    }

    public interface HostCallbacks {
        void finish();

        void setAccountAuthenticatorResult(String username, String authToken, String accountType);
    }

    @Override
    protected MainControllerUiCallbacks createUiCallbacks(MainControllerUi ui) {
        return new MainControllerUiCallbacks() {
            @Override
            public void onBottomMenuItemSelected(BottomMenuItem item) {
                Display display = getDisplay();
                if (display != null) {
                    showUiItem(display, item);
                }
            }
        };
    }

    public interface MainControllerUiCallbacks {

        void onBottomMenuItemSelected(BottomMenuItem item);
    }

    public interface MainControllerUi extends BaseUiController.Ui<MainControllerUiCallbacks> {
    }

    private void showUiItem(Display display, BottomMenuItem item) {
        Preconditions.checkNotNull(display, "display cannot be null");
        Preconditions.checkNotNull(item, "item cannot be null");
    }

    @Override
    protected void populateUi(MainControllerUi ui) {

    }

    public void setSelectedSideMenuItem(BottomMenuItem item) {
        mState.setSelectedSideMenuItem(item);
        populateUis();
    }

    public void attachDisplay(Display display) {
        Preconditions.checkNotNull(display, "display is null");
        Preconditions.checkState(getDisplay() == null, "we currently have a display");
        setDisplay(display);
    }

    @Override
    protected void onInited() {
        super.onInited();
        mIndexTabController.init();
        mUserController.init();
    }

    @Override
    protected void onSuspended() {
        super.onSuspended();
        mIndexTabController.suspend();
        mUserController.suspend();
    }

    public void detachDisplay(Display display) {
        Preconditions.checkNotNull(display, "display is null");
        Preconditions.checkState(getDisplay() == display, "display is not attached");
        setDisplay(null);
    } 
}

```



#### 1.2 View

``BaseActivity`` :

1. 持有``AndroidDisplay``、``MainController``，可以控制View间的跳转，控制所有Controller的状态；
2. handleIntent()，控制Activity指向的Fragment；
3. 关联``Display``到``BaseController``、设置所有``Controller``状态；

``` java
	public class BaseActivity extends ActionBarActivity implements MainController.HostCallbacks{

    private AndroidDisplay mDisplay;
    private MainController mMainController;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mDisplay = new AndroidDisplay(this);
        mMainController = App.from(this).getMainController();

        handleIntent(getIntent(),getDisplay());
    }

    protected final MainController getMainController() {
        return mMainController;
    }

    protected void handleIntent(Intent intent, Display display) {
    }

    public void setSupportActionBar(@Nullable Toolbar toolbar, boolean handleBackground) {
        setSupportActionBar(toolbar);
        getDisplay().setSupportActionBar(toolbar, handleBackground);
    }

    public Display getDisplay() {
        return mDisplay;
    }

    @Override
    protected void onResume() {
        super.onResume();
        mMainController.attachDisplay(mDisplay); 
        mMainController.init();

    }

    @Override
    protected void onPause() {
        mMainController.suspend(); 
        mMainController.detachDisplay(mDisplay);
        super.onPause();
    }

    @Override
    public void setAccountAuthenticatorResult(String username, String authToken, String accountType) {

    }
}
```

``BaseFragment``:

1. ​提供ToolBar、Display；

``` java
public abstract class BaseFragment extends Fragment {

    private Toolbar mToolbar;

    @Override
    public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
        mToolbar = (Toolbar) view.findViewById(R.id.toolbar);

        if (mToolbar != null) {
            setSupportActionBar(mToolbar);
        }
    }

    protected void setSupportActionBar(Toolbar toolbar) {
        setSupportActionBar(toolbar, true);
    }

    protected final void setSupportActionBar(Toolbar toolbar, boolean handleBackground) {
        ((BaseActivity)getActivity()).setSupportActionBar(toolbar, handleBackground);
    }

    protected Toolbar getToolbar() {
        return mToolbar;
    }

    protected Display getDisplay() {
        return ((BaseActivity) getActivity()).getDisplay();
    }

}
```