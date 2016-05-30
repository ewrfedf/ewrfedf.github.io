#### Dagger高级用法：任性inject

用了注入之后，就有这么一个需求：随时随地任性inject :), 对！我们将提供inject的注入，这样我们就可以实现任性inject！

开启我们的任性之旅吧

1. ##### 定义`Injector`接口
   
   ``` java
   public interface Injector {
       public void inject(Object object);
   }
   ```
   
   ​
   
2. ##### `Application`中实现`Injector`接口
   
   ``` java
   public class App extends Application implements Injector{
   
       private ObjectGraph applicationGraph;
   
       @Override
       public void onCreate() {
           super.onCreate();
           applicationGraph = ObjectGraph.create(getModules().toArray());
           applicationGraph.inject(this);
       }
   
       protected List<Object> getModules() {
           return Arrays.<Object>asList(new AppModule(this));
       }
   
       @Override
       public void inject(Object object) {
           applicationGraph.inject(object);
       }
   
   }
   ```
   
   ​
   
3. ##### 创建`InjectorModule`
   
   ``` java
   @Module(
           library = true
   )
   public class InjectorModule {
     
       private App app;
   
       public InjectorModule(App app) {
           this.app = app;
       }
   
       @Provides@Singleton
       public Injector providerInjector() {
           return app;
       }
   }
   ```
   
   ​
   
4. **关键的一步**：借助构造方法的参数随时随地获取`Injector`,并将含有注入的对象注入；
   
   ``` java
   @Singleton
   public class SingletonModule {
   
       private Injector injector;
   
       @Inject
       public SingletonModule(Injector injector) {
           this.injector = injector;
       }
   
       public void doToast() {
           InjectHandler handler = new InjectHandler();
           injector.inject(handler);
           handler.sendEmptyMessage(1);
       } 
   }
   ```
   
   ``` java
   public class InjectHandler extends Handler{
   
       @Inject
       Toast toast;
   
       @Override
       public void handleMessage(Message msg) {
           super.handleMessage(msg);
           toast.setText("InjectHandler");
           toast.show();
       }
   }
   ```
   
   > notice：在提供Toast的Provider的Module中加入injects = InjectHandler.class、在AppModule中includes =  InjectorModule.class、在ObjectGraph.create中加入new InjectorModule(this)
   
   ​
   
5. 调用
   
   ``` java
   public class MainActivity extends AppCompatActivity {
   
       @Inject
       SingletonModule sToast;
   
       @Override
       protected void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           setContentView(R.layout.activity_main2);
           ((App) getApplicationContext()).inject(this);
           sToast.doToast();
       }
   
   }
   ```
   
   ​

任性之旅已经结束，欢迎下次再来任性。

总结：

1. 任性并不是在任何地方，而是有一个前提条件需要@Singleton修饰Class， @Inject修饰构造方法，构造方法中就是一个获取Injector的机会，由此可见，任性只有一次机会；拿到Injector那就是任性的开始。
2. 继续任性的话只需要重复步骤4即可。



