#### Android Dagger用法

按照正常流程三步走：

1. 构建Module提供provider；
   
2. 将需要注入的class 加入需要待注册的类表中；
   
3. 将class的对象注入到ObjectGraph中，这样就可以获取到需要注入的对象； 
   
   ​ 

Dagger中还有一种间接的注入形式， 注入流程：

1. 构建Module：声明@Singleton以及@Inject修饰构造方法
   
   ``` java
   @Singleton
   public class SingletonModule {
   
       @Inject
       public SingletonModule() {
       }
   
       public void show(Context context){
           Toast.makeText(context,"SingletonModule",Toast.LENGTH_LONG).show();
       }
   }
   ```
   
2. 使用：声明、注入；

``` java
public class MainActivity extends AppCompatActivity {

    @Inject
    SingletonModule sToast;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main2);
        ((App) getApplicationContext()).inject(this);

        sToast.show(getApplication());
    }

}
```



总结：

1. ##### 构建依赖图
   
   主要是依赖``Module``间的依赖，以及要构建的对象，记录``provider``的相互依赖关系；
   
2. ##### ``@Singleton`` +``@Inject``
   
   这种方式不需要构建依赖图，只需要``inject``，就可以使用``Singleton``构建注入的对象；





dagger名词详解



``library = true``

表明这个Module提供的注入没有被明确的@Inject，而是通过其他的构造方法进行注入的，以此定义为lib类型的Module；



``include = {…class}``

表明这个Module中在提供依赖的同时还需要依赖其他的Module，这些其他的依赖需要在include中声明；还有另外一个比较有意思的用法：声明一个空的Module包含其相关联的几个Module，在ObjectGraph创建的时候只需要传入这个空的Module，就能完成其他的Module的注入；



