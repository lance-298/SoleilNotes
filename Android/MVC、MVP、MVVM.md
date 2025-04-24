## 1 MVC

MVC： Model - View - Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范，用一种业务逻辑、数据、界面显示分离的方法组织代码，将业务逻辑聚集到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。MVC被独特的发展起来用于映射传统的输入、处理和输出功能在一个逻辑的图形化用户界面的结构中。

* 模型层（Model）：针对业务建立的数据结构和相关的类；

* 视图层（View）：显示界面（XML 布局文件 或 Java 建立的界面）；

* 控制层（Controller）：Android 通常在 Activity、Fragment 中控制业务；

![](../asset/mvc.png)

Android 中 MVC 的缺点：

* View 层和 Model 层耦合，不利于维护。
* 实际开发中 Activity、Fragment 作为控制层，耦合了太多的业务， 往往可能有几千行代码。

## 2 MVP

MVP ：Model - View - Presenter，是 MVC 演化的版本，使用 MVP 时 View 和 Model 不再耦合，View 和 Model 通过 Presenter  进行交互。

![](../asset/mvp.png)

* Model：提供数据的存取功能，Presenter 通过 Model 获取、存储数据；
* View：负责处理用户事件和视图展示；
* Presenter：View 和 Model 沟通的桥梁，Model 获取的数据通过 Presenter 传递给 View；

### 3 MVVM

MVVM：Model - View - ViewModel，本质上是 MVC 层的演化版本，与 MVP 不同是，ViewModel 跟 Model 和 View 进行**双向绑定**，View 变化，ViewModel 就会通知 Model 数据改变，同样，Model 变化，ViewModel 也会通知 View。

![](../asset/mvvm.png)

缺点：

* 数据绑定使得 Bug 很难被调试；
* DataBinding 使用会生成大量的 Binding 类，可能影响编译速度；
* 数据双向绑定不利于代码重用；





```java
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">
    <data>
        <variable
            name="person"
            type="com.yoyiyi.test.mvvm.Person" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:orientation="vertical"
        android:layout_height="match_parent"
        tools:context=".MainActivity">
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{person.name}" />

    </LinearLayout>
</layout>


public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
        Person person = new Person("积极");
        binding.setP(person);
        
    }
 }   
```

### 3.1 基本用法

```java
//1.import 用法与别名
    <data>
        <import type="com.yoyiyi.test.mvvm.Person"
            alias="p1"/>
        <import type="com.yoyiyi.test.mvvm.alias.Person"
            alias="p2"/>
        
        <variable
            name="p1"
            type="p1" />
        <variable
            name="p2"
            type="p2" />
    </data>

//2.变量定义 java.lang.* 会被自动导入
   <data>       
        <variable
            name="name"
            type="String" />  
    </data>
    
//3.Converter 转换器
   <data>       
        <variable
            name="time"
            type="java.util.Date" />  
    </data>      
   
public class Utils {
    @BindingConversion
    public static String convertDate(Date date){
        return new SimpleDateFormat("yyyy-MM-dd").format(date);
    }

}
```

//4.双向绑定 @={}
Observable、ObservableField、集合类型 Observable 容器类


### 3.2ViewModel 与 Activity 生命周期对应关系
//![image](https://github.com/user-attachments/assets/9c9d9e81-86ef-44c9-a148-ee1033cafc5a)
//![image](https://developer.android.google.cn/static/codelabs/android-lifecycles/img/1d42e8efcb42ff58_1920.png)
![](../asset/viewmodel_1920.png)

一、 ViewModel 生命周期范围‌

ViewModel 的生命周期从首次通过 ViewModelProvider 获取实例开始，直到关联的 Activity ‌完全销毁‌（非配置变更导致的销毁）时结束46。
在 Activity 的 onCreate() 中首次获取 ViewModel 实例后，即使 Activity 因配置变更（如屏幕旋转）被销毁重建，ViewModel 仍会保留46。
‌关键生命周期回调的对应‌

‌onCreate()‌：建议在此处初始化并获取 ViewModel 实例47。
‌onDestroy()‌：
若 Activity 因用户主动退出（如返回键）或调用 finish() 被销毁，ViewModel 会触发 onCleared() 清理资源46。
若 Activity 因配置变更（如屏幕旋转）被销毁，ViewModel ‌不会‌被清除，重建的 Activity 会复用原实例48。
二、Activity 触发 ViewModel 结束的时机
ViewModel 的结束操作（即 onCleared() 调用）由以下场景触发：

‌Activity 被永久销毁‌
用户主动退出（如按下返回键）或代码中调用 finish()，导致 Activity 进入 onDestroy() 且不再重建46。
‌系统资源回收‌
当系统因内存不足需要回收后台 Activity 时，若 Activity 未被标记为可重建（如未设置 configChanges），ViewModel 会被销毁8。
‌ViewModel 的宿主范围结束‌
若 ViewModel 关联的 Activity 是唯一宿主，且宿主被永久销毁，ViewModel 随之结束46。
三、注意事项
‌避免内存泄漏‌
ViewModel ‌不应持有 Activity 的 Context‌，否则可能导致内存泄漏6。
‌数据恢复机制‌
在 Activity 因配置变更重建时，ViewModel 保留的数据可快速恢复 UI 状态，无需重新加载。
‌资源释放‌
在 onCleared() 中需手动释放资源（如取消异步任务、关闭数据库连接等）。

总结
ViewModel 的生命周期与 Activity 的解耦设计使其在配置变更时保持数据持久性，仅在 Activity 被永久销毁时释放资源。开发者需区分配置变更与永久销毁场景，合理管理数据与资源


