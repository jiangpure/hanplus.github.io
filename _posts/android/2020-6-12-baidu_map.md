---
layout:     post
title:      "『Android』 百度地图 SDK 的简单接入"
subtitle:   "简单快速地在你的项目中接入百度地图。"
date:       2018-04-29 12:00:00
author:     "purejiang"
header-img: "img/post-bg-1.jpg"
catalog: true
tags:
    - Android
---

## 一、 申请 API_KEY

#### 1. http://lbsyun.baidu.com/ :在这里注册并申请。注册之后出现应用列表，选择 *创建应用* ：

![创建应用](/img/android/baidu_map/0.jpg)

##### 2. 创建应用后如下图所示：

![选择](/img/android/baidu_map/1.jpg)

- 应用类型选择的话，我们选择 *Android SDK* 就可以了。

![SHA1](/img/android/baidu_map/2.jpg)

PS：`SHA1` 找了我好久...


- SHA1寻找之路：
左侧选择项目目录类型 *project* ，打开右侧 *Gradle* > *app*  > *android* > *signingReport* ，点击发现空白，突然发现左下角有个转换的键（左下标红的地方），答案出来了。

![](/img/android/baidu_map/3.jpg)

![](/img/android/baidu_map/4.jpg)

![](/img/android/baidu_map/5.jpg)


##### 3. 获取基本参数 

- 创建完成，获得 `API_KEY`。 

![](/img/android/baidu_map/6.jpg)


## 二、 下载百度定位 SDK

- 如无特殊要求，选择基础功能即可：

![](/img/android/baidu_map/7.jpg)


- 下载开发包解压后得到 *libs* 文件夹。

![](/img/android/baidu_map/8.jpg)


- *BaiduLBS_android.jar* 放在 *project* 目录下的 *app* > *libs* 目录下，其他 so 库放在 *app* > *main* > 新建文件夹 *jniLibs* 下，或者都可以放 *jniLibs* 下。


## 三、 使用百度地图

##### 1. 在 *AndroidManifest.xml* 中添加如下权限：

```
<!-- 这个权限用于进行网络定位-->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>

<!-- 这个权限用于访问 GPS 定位-->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>

<!-- 用于访问 wifi 网络信息，wifi 信息会用于进行网络定位-->
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>

<!-- 获取运营商信息，用于支持提供运营商信息相关的接口-->
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>

<!-- 这个权限用于获取 wifi 的获取权限，wifi 信息会用来进行网络定位-->
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>

<!-- 用于读取手机当前的状态-->
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>

<!-- 写入扩展存储，向扩展卡写入数据，用于写入离线定位数据-->
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>

<!-- 访问网络，网络定位需要上网-->
<uses-permission android:name="android.permission.INTERNET" />

<!-- SD卡读取权限，用户写入离线定位数据-->
<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
```

- 在 `application` 中声明 `service` ,每个APP拥有自己单独的定位`service`：

```
<service android:name="com.baidu.location.f" 
		 android:enabled="true" 
		 android:process=":remote"/>
```
 
- 百度定位 SDK 在4.2版本之后需要在 *AndroidMainfest.xml* 中正确设置 `Accesskey（AK）`，如果设置错误将会导致定位和地理围栏服务无法正常使用。设置 `AK`，在 `application` 中加入：

```
<meta-data
            android:name="com.baidu.lbsapi.API_KEY"
            android:value="AK" />       //AK:开发者申请的 Key
```

##### 2. 新建 `LBSwithBaidu`，`LBSwithBaidu` 代码如下：

```Java
public class LBSwithBaidu extends AppCompatActivity {
      private TextView text;
      private LocationClient client;
      private  StringBuilder connrentPosition;
      private MapView map;
      private BaiduMap baidumap;
      private boolean ismylocation=true;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //SDK的初始化
        SDKInitializer.initialize(getApplicationContext());
        //注册客户端定位的监听
        client = new LocationClient(getApplicationContext());
        client.registerLocationListener(new MylocationListener());

        setContentView(R.layout.activity_lbswith_baidu);
        text= (TextView) findViewById(R.id.textView_location);
        //获取地图控件的引用  
        map= (MapView) findViewById(R.id.mapwithbaidu);
        //获取地图实例
        baidumap=map.getMap();
        //设置是否在地图上显示位置
        baidumap.setMyLocationEnabled(true);
        //6.0以上动态权限的获取
        List<String> permission = new ArrayList<>();
        if (ContextCompat.checkSelfPermission(LBSwithBaidu.this, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
            permission.add(Manifest.permission.ACCESS_FINE_LOCATION);
        }
        if (ContextCompat.checkSelfPermission(LBSwithBaidu.this, Manifest.permission.READ_PHONE_STATE) != PackageManager.PERMISSION_GRANTED) {
            permission.add(Manifest.permission.READ_PHONE_STATE);
        }
        if (ContextCompat.checkSelfPermission(LBSwithBaidu.this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
            permission.add(Manifest.permission.WRITE_EXTERNAL_STORAGE);
        }
      
        if (!permission.isEmpty()) {
            String[] permissions = permission.toArray(new String[permission.size()]);
            ActivityCompat.requestPermissions(LBSwithBaidu.this, permissions, 1);

        } else {
            requestion();
        }

    }

    private void requestion() {
        initlocation();
        client.start();
    }

    private void initlocation() {
        LocationClientOption option=new LocationClientOption();
        option.setIsNeedAddress(true);
        option.setScanSpan(5000);
        client.setLocOption(option);
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        switch (requestCode) {
            case 1:
                if (grantResults.length > 0) {
                    for(int result:grantResults){
                        if(result!=PackageManager.PERMISSION_GRANTED){
                            Toast.makeText(this, "必须同意权限", Toast.LENGTH_SHORT).show();
                            finish();
                            return;
                        }
                    }
                    requestion();
                }else {
                    Toast.makeText(this, "发生错误", Toast.LENGTH_SHORT).show();
                }
                break;
            default:
        }
    }

    public void nagativato(BDLocation bdLocation) {
        if (ismylocation) {
            LatLng ll = new LatLng(bdLocation.getLatitude(), bdLocation.getLongitude());
            Log.d("pipa",bdLocation.getLatitude()+"::"+ bdLocation.getLongitude());
            MapStatusUpdate updata = MapStatusUpdateFactory.newLatLng(ll);
            baidumap.animateMapStatus(updata);
            updata = MapStatusUpdateFactory.zoomTo(16f);
            baidumap.animateMapStatus(updata);
            ismylocation = false;

        }
        MyLocationData.Builder loBuilder=new MyLocationData.Builder().latitude(bdLocation.getLatitude())
                .longitude(bdLocation.getLongitude());
        MyLocationData locationData=loBuilder.build();
        baidumap.setMyLocationData(locationData);

    }

    public class MylocationListener implements BDLocationListener {


        @Override
        public void onReceiveLocation(BDLocation bdLocation) {
           String name =  bdLocation.getLocType();   
           if(name==BDLocation.TypeGpsLocation||name==BDLocation.TypeNetWorkLocation){
                nagativato(bdLocation);
            }

            connrentPosition = new StringBuilder();
            connrentPosition.append("纬度:").append(bdLocation.getLatitude()).append("\n");
            connrentPosition.append("经度:").append(bdLocation.getLongitude()).append("\n");
            connrentPosition.append("国家:").append(bdLocation.getCountry()).append("\n");
            connrentPosition.append("省:").append(bdLocation.getProvince()).append("\n");
            connrentPosition.append("市:").append(bdLocation.getCity()).append("\n");
            connrentPosition.append("县:").append(bdLocation.getDistrict()).append("\n");
            connrentPosition.append("街道:").append(bdLocation.getStreet()).append("\n");
            connrentPosition.append("定位方式:");
            if (bdLocation.getLocType() == BDLocation.TypeGpsLocation) {
                connrentPosition.append("GPS");
            } else if (bdLocation.getLocType() == BDLocation.TypeNetWorkLocation) {
                connrentPosition.append("网络定位");
            }
            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    text.setText(connrentPosition);
                }
            });
        }
        @Override
        public void onConnectHotSpotMessage(String s, int i) {

        }
    }

    @Override
    protected void onResume() {
        super.onResume();
        //地图生命周期需实现
        map.onResume();
    }

    @Override
    protected void onPause() {
        super.onPause();
        //地图生命周期需实现
        map.onPause();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        client.stop();
        //地图生命周期需实现
        map.onDestroy();
        baidumap.setMyLocationEnabled(false);
    }
}
```

##### 3. 最终效果图：

![](/img/android/baidu_map/9.jpg)

如有不对之处，欢迎指正，谢谢~
