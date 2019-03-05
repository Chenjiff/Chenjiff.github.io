# 中山大学数据科学与计算机学院本科生实验报告
## （2018年秋季学期）
| 课程名称 | 手机平台应用开发 | 任课老师 | 郑贵锋 |
| :------: | :-------------: | :------------: | :-------------: |
| 年级 | 2016级 | 专业（方向） | 软件工程 |
| 学号 | 16340020 | 姓名 | 陈吉凡 |
| 电话 | 15017239913 | Email | 1004920224@qq.com |
| 开始日期 | 2018.10.18 | 完成日期 | 2018.10.24 |

---

## 一、实验题目
Broadcast 使用

---

## 二、实现内容
在第六周任务的基础上，实现静态广播、动态广播两种改变Notification 内容的方法。
  
 **要求** 
* 在启动应用时，会有通知产生，随机推荐一个食品。
* 点击通知跳转到所推荐食品的详情界面。
* 点击收藏图标，会有对应通知产生，并通过Eventbus在收藏列表更新数据
* 点击通知返回收藏列表。
* 实现方式要求:启动页面的通知由静态广播产生，点击收藏图标的通知由动态广播产生。
---

## 三、课堂及课后实验结果
### (1)实验截图
* 打开应用之后，通知栏弹出消息：  
![输入图片说明](https://images.gitee.com/uploads/images/2018/1025/190204_6d0ac7c3_2164918.png "11.png")  
* 点击这条消息，进入详情：  
![输入图片说明](https://images.gitee.com/uploads/images/2018/1025/190237_8b7e808d_2164918.png "22.png")  
* 点击收藏按钮，通知栏弹出新消息：  
![输入图片说明](https://images.gitee.com/uploads/images/2018/1025/190301_9a85824d_2164918.png "33.png")  
* 点击该条消息，可以查看收藏夹界面：  
![输入图片说明](https://images.gitee.com/uploads/images/2018/1025/190355_b89d400e_2164918.png "44.png")  
  
### (2)实验步骤以及关键代码
首先写出静态广播和动态广播的receiver，静态的：
```
public class StaticReceiver extends BroadcastReceiver {
    private static final String STATIC_ACTION = "com.code.chenjifff.experimentone.StaticFilter";
    //下面的方法为接收到广播后执行的动作
    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    @Override
    public void onReceive(Context context, Intent intent) {
        if(intent.getAction().equals(STATIC_ACTION)){
            Bundle bundle = intent.getExtras();
            //通知部分
        }
    }
}
```
动态的类似，但是注册方法不同，静态注册方法：在AndroidManifest.xml文件中加入：
```
<receiver android:name=".StaticReceiver">
            <intent-filter>
                <action android:name="com.code.chenjifff.experimentone.StaticFilter"/>
            </intent-filter>
</receiver>
```
动态的：在发送广播的类中加入：
```
IntentFilter filter = new IntentFilter(DYNAMIC_ACTION);
DynamicReceiver dynamicReceiver = new DynamicReceiver();
registerReceiver(dynamicReceiver, filter);
```
然后发送广播，静态的（动态类似）：
```
Intent intentStaticBroadcast = new Intent(STATIC_ACTION);
Bundle bundleBroadcast = new Bundle();
Random random = new Random();
int i = random.nextInt(foodData.size());
bundleBroadcast.putSerializable("food", foodData.get(i));
intentStaticBroadcast.putExtras(bundleBroadcast);
sendBroadcast(intentStaticBroadcast);
```
接着是完善通知功能：
```
Food food = (Food)bundle.getSerializable("food");
//创建PendingIntent以便点击导向食品详情
Intent foodIntent = new Intent(context, FoodDetail.class);
foodIntent.putExtras(bundle);
PendingIntent pendingIntent = PendingIntent.getActivity(context, 0, foodIntent, 0);
//发送通知消息
Notification.Builder builder = new Notification.Builder(context);
builder.setContentTitle("今日推荐")
        .setContentText(food.getName())
        .setTicker("你有一条新消息")
        .setSmallIcon(R.mipmap.empty_star)
        .setContentIntent(pendingIntent)
        .setAutoCancel(true);
NotificationManager notificationManager = 
    (NotificationManager)context.getSystemService(context.NOTIFICATION_SERVICE);
Notification notification = builder.build();
notificationManager.notify(0, notification);
```
使用EventBus完善更新收藏夹功能，在收藏夹界面的类中注册EventBus：  
```
EventBus.getDefault().register(this);
```
订阅事件，CollectEvent为自己创建的类，用于传递食品信息：
```
@Subscribe(threadMode = ThreadMode.MAIN)
    public void Event(CollectEvent collectEvent) {
        //更新收藏夹的代码
    }
```
记得注销：
```
@Override
    protected void onDestroy() {
        super.onDestroy();
        if(EventBus.getDefault().isRegistered(this)) {
            EventBus.getDefault().unregister(this);
        }
    }
```
  
### (3)实验遇到的困难以及解决思路
* 在使用EventBus来实现更新收藏夹时，发现收藏夹里的每个食物对象虽然名字和食物列表界面的一样，但是实际是不同的类，导致判断某种食品是否在收藏夹中不正确，推测是`bundle.putSerializable("food", food);`的问题，暂时解决方法是遍历收藏食品列表并判断其中是否有相同name属性的元素，以后想到更好的解决方法在更新。  
更新：原因：经过putSerializable之后，另一activity收到的是内容相同但是内存地址不同的类，而在EventBus处理时，传递的就是这个类，不会影响内存地址，放入collectFoodData也就是这个类，而Food类没有重写equals函数，contains比较的也就是类地址了，自然与原来的不同。解决：在Food里重写equals函数，原理和上面的暂时解法差不多，但是更简便易懂。总结：对一个类进行序列化会改变地址，而Eventbus等activity之间的交互不会。

---
## 四、实验思考及感想
在这次实验中，我加深了对安卓应用广播机制的理解，并初步学会了它的使用方法，学会这种在日常生活中经常会接触并且有实用价值的代码知识，让我感觉收获很不错，但是在本次实验中，我发现我对不同组件之间跳转的机制不是很清楚，以后必须弥补回来，争取理解透彻。另外，这次还收获了在AndroidStudio下使用debug工具的技巧，再加上看源码，debug能力增长了许多。

---

#### 作业要求
* 命名要求：学号_姓名_实验编号，例如12345678_张三_lab1.md
* 实验报告提交格式为md
* 实验内容不允许抄袭，我们要进行代码相似度对比。如发现抄袭，按0分处理