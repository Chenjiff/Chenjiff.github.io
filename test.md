# 中山大学数据科学与计算机学院本科生实验报告
## （2018年秋季学期）
| 课程名称 | 手机平台应用开发 | 任课老师 | 郑贵锋 |
| :------: | :-------------: | :------------: | :-------------: |
| 年级 | 2016级 | 专业（方向） | 软件工程 |
| 学号 | 16340020 | 姓名 | 陈吉凡 |
| 电话 | 15017239913 | Email | 1004920224@qq.com |
| 开始日期 | 2018.11.25 | 完成日期 | 2018.12.05 |

## 一、实验题目
### 简单音乐播放器

---

## 二、实现内容
实现一个简单的播放器，要求功能有：  
<table>
    <tr>
        <td ><img src="https://gitee.com/code_sysu/PersonalProject4/raw/master/manual/images/fig1.jpg" >打开程序主页面</td>
        <td ><img src="https://gitee.com/code_sysu/PersonalProject4/raw/master/manual/images/fig2.jpg" >开始播放</td>
    </tr>
    <tr>
        <td ><img src="https://gitee.com/code_sysu/PersonalProject4/raw/master/manual/images/fig3.jpg" >暂停</td>
        <td ><img src="https://gitee.com/code_sysu/PersonalProject4/raw/master/manual/images/fig1.jpg" >停止</td>
    </tr>
</table>

1. 播放、暂停、停止、退出功能，按停止键会重置封面转角，进度条和播放按钮；按退出键将停止播放并退出程序
2. 后台播放功能，按手机的返回键和home键都不会停止播放，而是转入后台进行播放
3. 进度条显示播放进度、拖动进度条改变进度功能
4. 播放时图片旋转，显示当前播放时间功能，圆形图片的实现使用的是一个开源控件CircleImageView
5. 在保持上述原有的功能的情况下，使用rxJava代替Handler进行UI的更新。

**附加内容（加分项，加分项每项占10分）**

* 选歌

用户可以点击选歌按钮自己选择歌曲进行播放，要求换歌后不仅能正常实现上述的全部功能，还要求选歌成功后不自动播放，重置播放按钮，重置进度条，重置歌曲封面转动角度，最重要的一点：需要解析mp3文件，并更新封面图片。

---

## 三、课堂实验结果
### (1)实验截图
* 初始状态：  
![输入图片说明](https://images.gitee.com/uploads/images/2018/1207/224649_bae0738d_2164918.png "1.png")  
* 点击播放键：
![输入图片说明](https://images.gitee.com/uploads/images/2018/1207/224658_7e63989b_2164918.png "2.png")  
* 暂停:  
![输入图片说明](https://images.gitee.com/uploads/images/2018/1207/224709_3f27b417_2164918.png "3.png")  
* 停止：  
![输入图片说明](https://images.gitee.com/uploads/images/2018/1207/224719_b89a31e7_2164918.png "4.png")  

### (2)实验步骤以及关键代码
1. 实现MediaPlayer的关键代码，注意要提前获取文件的读写权限：
```
//获取权限
int permission = ActivityCompat.checkSelfPermission(MainActivity.this, "android.permission.WRITE_EXTERNAL_STORAGE");
if (permission != PackageManager.PERMISSION_GRANTED) {
    ActivityCompat.requestPermissions(MainActivity.this, PERMISSIONS_STORAGE, 1);
}
```
```
        //实例一个MediaPlayer
        mediaPlayer = new MediaPlayer();
        try {
            mediaPlayer.setDataSource(Environment.getExternalStorageDirectory() + "/Music/山高水长.mp3");
            mediaPlayer.prepare();
            mediaPlayer.seekTo(0);
            mediaPlayer.setLooping(true);
        }
        catch (Exception e) {
            e.printStackTrace();
        }
```
播放、暂停、停止以及获取当前进度等等也是简单地调用MediaPlayer的接口即可。 
 
2. 创建Service类，实现后台播放，主要有以下：
```
//重写onBind函数，在里面进行MediaPlayer的创建，这个函数在Activity绑定服务时调用
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        //创建MediaPlayer
        return musicBinder;
    }
```
```
//自己实现一个Binder类用于与Activity的交互
    public MusicBinder musicBinder = new MusicBinder();

    public class MusicBinder extends Binder {
        @Override
        protected boolean onTransact(int code, @NonNull Parcel data, @NonNull Parcel reply, int flags) throws RemoteException {
            switch (code) {
                //根据不同code做出不同响应
            }
            return super.onTransact(code, data, reply, flags);
        }
    }
```
3.用Handle实现UI的更新：
```
//创建线程来获取更新UI所需的数据
    class PlayThread extends Thread {
        @Override
        public void run() {
            Parcel data = Parcel.obtain();
            Parcel reply = Parcel.obtain();
            try {
                iBinder.transact(4, data, reply, 0);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
            int currentMsc = reply.readInt();
            Message msg = handler.obtainMessage(1);
            msg.arg1 = currentMsc;
            handler.sendMessage(msg);
        }
    }
```
```
//创建Handle并重写
    private Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            switch(msg.what) {
                case -1:
                    handler.removeCallbacksAndMessages(null);
                    break;
                default:
                    seekBar.setProgress(msg.arg1);
                    circleImageView.setRotation(circleImageView.getRotation() + 0.24f);
                    handler.postDelayed(playThread, 40);
                    currentDuration.setText(simpleDateFormat.format(msg.arg1));
            }
        }
    };
```


### (3)实验遇到的困难以及解决思路
1. LinearLayout的layout_gravity问题，需要注意在这个布局下，layout_gravity往往只对和布局的垂直方向有效。
2. 在setDataSource上出了许多bug。首先是Android6.0要求必须动态申请权限，并且MediaPlayer要求读和写权限，仅有读权限是不可以的，并且Android手机存储方面，要理清内部存储、外部存储的关系。  
3.removeCallbacks有时会失效。经过查阅和实验，发现是由于这个函数无法中止已运行的线程，而导致可能会出现线程运行至一半而又刚好调用这个函数，表面看起来就失效了。  
4.service的存在问题，service可能存在应用的进程，也可能属于独自的进程，主要与AppManifest.xml文件中的定义方式有关系。  
---

## 五、实验思考及感想
这次实验不仅让我对安卓系统下线程的一些机制有了很多认识，而且也补充了以前关于线程的一些知识点。最后还使用了service这个经常有接触到的东西，通过自己的实践也加深了理解，本次实验收获颇多，受益匪浅。

---

#### 作业要求
* 命名要求：学号_姓名_实验编号，例如12345678_张三_lab1.md
* 实验报告提交格式为md
* 实验内容不允许抄袭，我们要进行代码相似度对比。如发现抄袭，按0分处理