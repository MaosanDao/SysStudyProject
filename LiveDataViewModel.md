# 如何使用LiveData和ViewModel
## LiveData
    本章节主要讲怎么使用LiveData
## ViewModel
### ViewModel的创建方法
创建类直接继承**ViewModel**即可
```kotlin
class SecondViewModel : ViewModel() {
    //LiveData数据的创建
    var userData: MutableLiveData<UserInfo> = MutableLiveData()

    /**
     *  模拟获取数据
     */
    fun getUserInfo() {
        val user = UserInfo("李四",(1000..5000).random())
        userData.postValue(user)
    }
}
```
在Activity中进行绑定和使用
```kotlin
class SecondActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContentView(R.layout.activiry_second)
        //进行绑定
        val secondViewModel = ViewModelProvider(this).get(SecondViewModel::class.java)
        secondViewModel.userData.observe(this, Observer {
            //数据发生变化的时候，通知UI进行更新
            mTvShow.text = "姓名：${it.name} \n薪水：${it.salary}"
        })

        /**
         * 模拟数据源更新
         * 这里更新了，上面的观察者即可实时通知UI进行变化
         */
        mBtnData.setOnClickListener {
            secondViewModel.getUserInfo()
        }
    }
}
```
### 数据转换
Transformations的map使用
```kotlin
Transformations.map(secondViewModel.userData, object : Function<UserInfo, UserInfo> {
    override fun apply(userInfo: UserInfo): UserInfo {
        //将name的属性进行了再次的赋值
        userInfo.name = "张三"
        return userInfo
    }

}).observe(this, Observer {
    mTvShow.text = "姓名：${it.name} \n薪水：${it.salary}"
})
```
Transformations的switchMap使用

有些数据需要依赖其他数据进行查询
```kotlin
Transformations.switchMap(secondViewModel.userData, object : Function<UserInfo, LiveData<UserInfo>> {
        override fun apply(userInfo: UserInfo): LiveData<UserInfo> {
            return secondViewModel.getUserName(userInfo)
        }
    }).observe(this, Observer {
    mTvShow.text = "姓名：${it.name} \n薪水：${it.salary}"
})

//在ViewModel中定义getUserName方法
fun getUserName(userInfo: UserInfo): LiveData<UserInfo> {
    userInfo.name = userInfo.name + "switchMap"
    switchMapData.value = userInfo
    return switchMapData
}
```
### MediatorLiveData
当我们页面需要多个不同的数据源的时候，如果我们都是单独的使用LiveData，会导致Activity中定义很多observe，出现很多多余的代码。MediatorLiveData就为解决这个问题的。它可以将多个LiveData合并在一起，只需要定义一次observe就可以。
```kotlin
var data1: MutableLiveData<UserInfo> = MutableLiveData()
var data2: MutableLiveData<UserInfo> = MutableLiveData()
var mediatorLiveData: MediatorLiveData<UserInfo> = MediatorLiveData()

val user1 = UserInfo("李四1", (1000..5000).random())
val user2 = UserInfo("李四2", (1000..5000).random())

data1.postValue(user1)
data2.postValue(user2)

mediatorLiveData.addSource(data1, object : Observer<UserInfo> {
    override fun onChanged(info: UserInfo) {
        mediatorLiveData.value = info
    }

})

mediatorLiveData.addSource(data2, object : Observer<UserInfo> {
    override fun onChanged(info: UserInfo) {
        mediatorLiveData.value = info
    }

})

//在Activity中进行使用
secondViewModel.mediatorLiveData.observe(this, Observer {
    //因为两个数据是绑在一起的，所以这里通过name这个属性来进行判断
    if (it.name.contains("1")) {
        mTvShow.text = "姓名：${it.name} \n薪水：${it.salary}"
    } else {
        mTvShowOther.text = "姓名：${it.name} \n薪水：${it.salary}"
    }

})

```

































