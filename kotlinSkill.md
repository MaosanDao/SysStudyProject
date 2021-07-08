# Kotlin中使用单例模式
在Kotlin中有个天然特性可以支持线程安全DCL的单例，可以说也是非常非常简单，就仅仅3行代码左右，那就是Companion Object + lazy属性代理。
```kotlin
class KLazilyDCLSingleton private constructor() : Serializable {//private constructor()构造器私有化

    fun doSomething() {
        println("do some thing")
    }

    private fun readResolve(): Any {//防止单例对象在反序列化时重新生成对象
        return instance
    }

    companion object {
        //通过@JvmStatic注解，使得在Java中调用instance直接是像调用静态函数一样，
        //类似KLazilyDCLSingleton.getInstance(),如果不加注解，在Java中必须这样调用: KLazilyDCLSingleton.Companion.getInstance().
        @JvmStatic
        //使用lazy属性代理，并指定LazyThreadSafetyMode为SYNCHRONIZED模式保证线程安全
        val instance: KLazilyDCLSingleton by lazy(LazyThreadSafetyMode.SYNCHRONIZED) { KLazilyDCLSingleton() }
    }
}
```
# Kotlin中无法解析空构造函数的问题
### 出现问题
```xml
com.alibaba.fastjson.JSONException: default constructor not found
```
### 如何解决
```
//解决kotlin无法解析空构造函数的问题
implementation "org.jetbrains.kotlin:kotlin-reflect:$kotlin_version"
```
