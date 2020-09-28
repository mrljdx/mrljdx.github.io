# 自定义Mob授权页面注意事项

现在很多应用都需要加入三方账号登陆,最常见的微博微信登陆等,虽然微博和微信这两大平台有提供给开发者三方登陆的jar包,但是个人建议集成Mob或其他统计平台提供的sdk.
原因有两点:
1. 便于集成,调用的授权方法统一,不需要花费时间去研究其他平台的sdk相关的授权代码;
2. 后台数据统计报表,便于分析用户组成,以及授权用户的平台占比.

在平时开发中,为了保证App风格的统一需要根据需求修改Mob(原ShareSDK)的授权界面UI.
官方修改UI介绍:
> http://bbs.mob.com/thread-278-1-1.html

<!--more-->

下面介绍一下在实际运用中集成的问题:

## 首先了解Mob授权页的组成

![Mob授权页](http://bbs.mob.com/data/attachment/forum/201410/24/140725xijxlp0p5393hpi9.png)

## 修改授权页代码

    /**
     * User: Mrljdx(mrljdx@qq.com)
     * Date: 2015-07-25
     * Time: 13:59
     * 注意，在注册此类时，需要完整路径（包名+类名 ）
     */
    public class MyAuthPageAdapter extends AuthorizeAdapter {
    
        @Override
        public void onCreate() {
            Context context = mApplication.getInstance().getApplicationContext(); //获取上下文
            hideShareSDKLogo();//隐藏logo
            disablePopUpAnimation(); //屏蔽弹框动画
            TitleLayout llTitle = getTitleLayout(); //获取授权页的title
            llTitle.setBackgroundResource(R.color.header_bg_color); //设置授权页titleBar的背景为App的主题背景
            ImageView ivBack = llTitle.getBtnBack(); //获取返回按键图标
            ivBack.setImageResource(R.drawable.ic_back);//设置返回图标
            llTitle.getChildAt(1).setVisibility(View.GONE); //去掉分割线
            TextView titleView = llTitle.getTvTitle(); //获取标题
            titleView.setText(""); //设置标题
            titleView.setTextColor(Color.parseColor("#656565"));//设置标题字体颜色
            titleView.setGravity(Gravity.CENTER); //设置居中显示
            titleView.setTypeface(null, Typeface.NORMAL); //设置字体
            titleView.setHeight(dip2px(context, 50)); //注意:这里setHeight为像素,需要将DP转换为PX
            titleView.setPadding(0, 0, 50, 0); //设置显示的title居中显示
        }
    
        /**
         * 根据手机的分辨率从 dp 的单位 转成为 px(像素)
         */
        private int dip2px(Context context, float dpValue) {
            final float scale = context.getResources().getDisplayMetrics().density;
            return (int) (dpValue * scale + 0.5f);
        }    

    }

## 修改AndroidMainfest.xml
在AndroidMainfest.xml添加如下代码:

    <activity
        android:name="com.mob.tools.MobUIShell"
        android:configChanges="keyboardHidden|orientation|screenSize"
        android:screenOrientation="portrait"
        android:theme="@android:style/Theme.Translucent.NoTitleBar"
        android:windowSoftInputMode="stateHidden|adjustResize">
        <intent-filter>
            <data android:scheme="tencent100371282" />
            <action android:name="android.intent.action.VIEW" />
    
            <category android:name="android.intent.category.BROWSABLE" />
            <category android:name="android.intent.category.DEFAULT" />
        </intent-filter>
        <!--注意:如果只是 .MyAuthPagerAdapter 则会导致修改UI无效-->
        <meta-data
             android:name="AuthorizeAdapter"
             android:value="包名+MyAuthPageAdapter路径" />
    </activity>

## 注意事项

1. 自定义授权页面类时,注意在TitleBar的高度的单位是项目,需要将你定义的TitlBar的dp值转换为px;
2. 在AndroidMainfest.xml中注册此类时,应填写"包名"+"类名"的完整路径;
3. 别在此类中定义回调监听函数,因为没什么用.

## END
参考资料:http://bbs.mob.com/thread-278-1-1.html
基本上一个App集成Mob的分享SDK后,只有修改授权页面的需求,如果有其他需求,请去官方BBS上找资料.

---------------------
