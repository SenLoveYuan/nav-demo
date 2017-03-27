# android-trip
基于高德地图Android API快速搭建用车解决方案的示例

## 前述 ##
- [高德官网申请Key](http://lbs.amap.com/dev/#/).
- 阅读[参考手册](http://a.amap.com/lbs/static/unzip/Android_Map_Doc/index.html).
- 工程基于Android 3D地图SDK实现

## 扫一扫安装##
![Screenshot]( https://raw.githubusercontent.com/amap-demo/android-trip/master/resource/download.png)  

##出行类解决方案——乘客端
## 使用方法##
###1:配置搭建AndroidSDK工程###
- [Android Studio工程搭建方法](http://lbs.amap.com/api/android-sdk/guide/creat-project/android-studio-creat-project/#add-jars).
- [通过maven库引入SDK方法](http://lbsbbs.amap.com/forum.php?mod=viewthread&tid=18786).

###接口使用###

搭建出行类用车乘客端的LBS部分，仅需要以下三步：
###Step 1：定位+逆地理编码+地图打点，搞定前端展示
效果如下图。屏幕中间的蓝色标记 用到了定位功能。位于顶部的输入框 用到了逆地理编码功能；地图上蓝色的出租车标识 运用了地图打点。

![Screenshot](https://raw.githubusercontent.com/amap-demo/android-trip/master/resource/mark.png)   

###定位
需要做的仅仅是将libs下的库拷贝到你的工程中，打开高小德用车的源代码，将LocationTask.java拷贝到工程中，找到MainActivity.java文件，仿照其中的写法，在合适的位置调用如下这几行，即可调起定位功能。
``` java
private LocationTask mLocationTask;//声明定位对象
mLocationTask = LocationTask.getInstance(getApplicationContext());//获取定位单例
mLocationTask.setOnLocationGetListener(locationTaskListener);//设置监听器
mLocationTask.startSingleLocate();//开始定位
mLocationTask.onDestroy();//销毁定位资源
```
###逆地理编码

先行将RegeocodeTask.java拷贝到工程中，依旧参照MainActivity.java文件，在合适的位置调用如下的代码段，即可调起逆地理编码功能。
``` java
private RegeocodeTask mRegeocodeTask;//声明逆地理编码对象
mRegeocodeTask = new RegeocodeTask(getApplicationContext());//为该对象赋值
mRegeocodeTask.setOnLocationGetListener(regeocodeTaskListener);//设置监听器
mRegeocodeTask.search(mPosition.latitude, mPosition.longitude);//启动逆地理编码服务
```
###地图打点

Utils.java封装了在地图上打点的功能，如下调用即可。
``` java
Utils.addEmulateData(mAmap, mStartPosition);//调用addEmulateData方法。参数解释：参数一为地图的controller；参数二为模拟数据的中心点，真实数据可以传递数据列表。
```
###Step 2 输入提示+POI搜索，搞定目的地
完成了以上这三大要点之后，接下来需要告诉司机师傅乘客要去哪里了。如下图所示，为了给用户提供一个更好的用车体验，在支持录入目的地的功能基础上，提供输入提示+POI搜索功能，进一步提升体验。
![Screenshot](https://raw.githubusercontent.com/amap-demo/android-trip/master/resource/search.png)   

###输入提示

拷贝InputTipTask.java到创建的工程中，查看DestinationActivity.java示例，实现TextWatcher接口后，进行如下调用：
``` java
//在onTextChanged方法中调用InputTipTask 的getInstance方法
@Override
public void onTextChanged(CharSequence s, int start, int before, int count) {
    if(RouteTask.getInstance(getApplicationContext()).getStartPoint()==null){
        Toast.makeText(getApplicationContext(), "检查网络，Key等问题", Toast.LENGTH_SHORT).show();
        return;
    }
    InputTipTask.getInstance( mRecomandAdapter).searchTips(getApplicationContext(),s.toString(),
            RouteTask.getInstance(getApplicationContext()).getStartPoint().city);
}
```	
输入提示功能完成！

###POI搜索

拷贝PoiSearchTask.java到创建的工程中，参考DestinationActivity.java调用核心方法的方式，即可实现如图所示的POI搜索。核心方法如下：
``` java
PoiSearchTask poiSearchTask=new PoiSearchTask(getApplicationContext(),
mRecomandAdapter);//生成poiSearchTask对象
poiSearchTask.search(mDestinaionText.getText().toString(),RouteTask.getInstance(getApplicationContext()).getStartPoint().city);//开始进行POI搜索
```
###Step 3 驾车路径规划，搞定费用预估

驾车路径规划功能提供了费用预估功能，效果参照下图
![Screenshot](https://raw.githubusercontent.com/amap-demo/android-trip/master/resource/result.png)    

###驾车路径规划

拷贝RouteTask.java文件到创建的工程中，参照MainActivity.java文件中的示例代码，实现OnRouteCalculateListener接口。调用的核心方法如下：
``` java
@Override 
//OnRouteCalculateListener提供的onRouteCalculate回调函数，高小德用车已经将返回onRouteCalculate结果的代码完整封装
public void onRouteCalculate(float cost,float distance,int duration) {
	mDestinationContainer.setVisibility(View.VISIBLE);

	mRouteCostText.setVisibility(View.VISIBLE);
	mDesitinationText.setText(RouteTask
			.getInstance(getApplicationContext()).getEndPoint().address);
	mRouteCostText.setText(
			String.format("预估费用%.2f元，距离%.1fkm,用时%d分", cost,distance,duration)
			 );
	mDestinationButton.setText("我要用车");
	mCancelButton.setVisibility(View.VISIBLE);
	mDestinationButton.setOnClickListener(null);
}
```