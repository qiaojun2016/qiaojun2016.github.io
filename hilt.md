# Hlit 依赖注入

## 说在前面的话
为了实现对象的创建与使用分开，所以我们需要依赖注入；让框架为我生成我们所需要的对象，我们直接在app中使用。hilt或者dagger2就是一个这种的框架。这两个框架都有一个容器的概念，容器就是框架用来管理对象的地方，不同的容器它的生命周期也有所不同。

hilt模块是创建对象的地方，告诉容器这个对象应该如何创建。一般的我们可以使用构造函数注入对象，这样就用不到module，hilt自己就能找到这个对象然后创建。而对于不能使用构造函数创建，类似于第三方的类库。那就需要我们为其创建一个Module。利用注解来创建。



例如我需要一个对象它在整个应用可用，而且是单例。那么我们就要把它放到ApplicationComponent中。
```kotlin
@InstallIn(SingletonComponent::class)
@Module
class AppModule {
    @Singleton
    @Provides
    fun provideRetrofit():Retrofit {
        return Retrofit.Builder()
            .baseUrl("")
            .addConverterFactory(GsonConverterFactory.create())
            .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
            .client(client)
            .build(); 
    }
}
```
以上，就是告诉hilt我要创建一个全局唯一的 Retrofit，让 **SingletonComponent** 容器 管理(installIn)它生命周期。假如你不写`@Singleton` 那么每次使用时候就会创建一个新Retrofit实例。

使用是指这样使用
```java
// 不使用@SingleTon
@Inject late var retrofit: Retrofit
```
但是有时候，我们希望我们对象与某个生命周期组件（Activity、Fragmnet、View）绑定，在这个对象存活时候保持其单例特性。


```java
// 
@InstallIn(ActivityComponent::class)
@Module
class LoginModule {
    @ActvityScope
    @Provides
    fun provideLoginRepository()
}
```
`@ActvityScope` 表示这个对象在 **ActivityComponent** 容器声明周期内保持单例特性。如果不加，那么每次使用时候都会创建新的对象。

## 我们看一下 hilt用的注解
### `@HiltAndroidApp`
官方文档定义
> All apps that use Hilt must contain an Application class that is annotated with @HiltAndroidApp.
### `@AndroidEntryPoint`
一些Android系统的类，无法手动创建，所以需要hilt在内部进行注入对象。但是hilt要知道为那些系统对象创建。所以就要为需要注入对象的系统类打个标记

```kotlin
@AndroidEntryPoint
public class ExampleActivity extends AppCompatActivity { ... }
```
### `@Inject`
构造函数注入
```kotlin
class AnalyticsAdapter @Inject constructor(
  private val service: AnalyticsService
) { ... }
```
hilt就会知道， AnalyticsAdapter 怎么创建了。大多数情况下我们自己创建的类，都可以使用构造函数方式进行注入。


### `@Module`
创建一个hilt模块，因为不是所有的类型都可以直接在实现类上利用`@Inject`注入。比如第三方类，数据库等。
在模块中创建对象有两种注解方式
#### `@Provides`
#### `@Binds`

## 容器
hilt 管理对象生命周期的地方。
### `SingletonComponent`
### `ActivityComponent`





