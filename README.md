# 튜토리얼 > Android용 지도 시작하기

이 문서는 안드로이드 플랫폼에서 애플리케이션 개발에 대한 기본적인 지식을 갖춘 사용자를 대상으로 안드로이드용 네이버 지도 라이브러리 사용법에 대해서 설명합니다.  
안드로이드 애플리케이션에서 네이버 지도 라이브러리를 사용하기 위해서는 아래 2가지 요소를 준비해야 합니다.

1. 안드로이드 네이버 지도 라이브러리용 API 키 발급
2. 안드로이드용 네이버 지도 라이브러리 (nmaps.jar : 샘플 프로젝트에 포함)

발급받은 API 키는 키 발급 요청 시 등록한 패키지 네임과 동일한 애플리케이션에서만 정상적으로 작동합니다.   
즉, API 키 발급을 요청하기 전에 지도 라이브러리를 사용할 애플리케이션의 패키지 네임을 먼저 결정해야 합니다.   
애플리케이션의 패키지 네임은 Context.getPackageName() 메서드로 확인할 수 있습니다.  

### Table of Contents
1. [지도 생성 사용 예제](#id-section1)
2. [지도 위에 오버레이 아이템 표시](#id-section2)
3. [지도 위에 경로 그리기](#id-section3)
4. [지도 위 오버레이 아이템 위치 이동](#id-section4)
5. [현재 위치 표시 및 나침반에 의한 지도 회전](#id-section5)
6. [좌표를 주소로 변환 (Reverse geo-coding)](#id-section6)

<div id='id-section1'/>
## 지도 생성 사용 예제
안드로이드 애플리케이션에서 네이버 지도 라이브러리를 사용하여 지도를 표시하는 방법을 설명합니다.  

__1) 애플리케이션 프로젝트를 생성합니다.__  
지도 라이브러리용 API 키 발급 요청 시 등록한 패키지 네임과 동일한 프로젝트를 생성합니다.

__2) 프로젝트에 지도 라이브러리를 추가합니다.__  
지도 라이브러리(nmaps.jar) 파일을 프로젝트의 외부 라이브러리로 설정합니다.

__3) AndroidManifest 파일에 아래와 같이 네트워크 접근 권한을 설정합니다.__  
지도 라이브러리에서 지도 타일 다운로드 및 서버 API 연동을 위해서 네트워크 연결이 필요합니다.  
```xml
<uses-permission android:name="android.permission.INTERNET"/>
```
__4) NMapActivity 클래스를 상속받은 Activity 클래스를 생성합니다.__   
NMapView 클래스를 포함하는 Activity는 반드시 NMapActivity클래스를 상속받아야 하며 
해당 Activity는 하나의 NMapView 클래스만 포함해야 합니다.  

__5) NMapActivity클래스를 상속받은 Activity의 onCreate() 메서드에서 NMapView객체를 생성합니다.__  
NMapView 객체는 XML layout을 이용해서 생성하거나 소스 코드 상에서 직접 생성할 수 있습니다. 
NMapView 객체를 생성한 후, 아래와 같이 발급받은 API 키를 설정합니다.  
```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    // create map view
    mMapView = new NMapView(this);

    // set a registered API key for Open MapViewer Library
    mMapView.setApiKey(API_KEY);

    // set the activity content to the map view
    setContentView(mMapView);

    // initialize map view
    mMapView.setClickable(true);

    // register listener for map state changes
    mMapView.setOnMapStateChangeListener(onMapViewStateChangeListener);
    mMapView.setOnMapViewTouchEventListener(onMapViewTouchEventListener);

    // use map controller to zoom in/out, pan and set map center, zoom level etc.
    mMapController = mMapView.getMapController();
}
```
그리고 NMapView 객체에서 발생하는 상태 변화 및 터치 이벤트에 대한 콜백 인터페이스를 등록합니다. 지도 초기화가 완료되면 아래의 콜백 인터페이스가 호출됩니다. 초기화가 성공하면 지도 보기 모드 및 중심 좌표 등을 설정하고 실패하면 적절한 예외 처리를 수행합니다
```java
public void onMapInitHandler(NMapView mapView, NMapError errorInfo) {
    if (errorInfo == null) { // success
        mMapController.setMapCenter(new NGeoPoint(126.978371, 37.5666091), 11);
    } else { // fail
        Log.e(LOG_TAG, "onMapInitHandler: error=" + errInfo.toString());
    }
}
```
마지막으로, 지도를 확대하거나 축소할 때 내장된 컨트롤러를 사용하려면 아래와 같이 설정합니다.
```java
// use built in zoom controls
mMapView.setBuiltInZoomControls(true, null);    
```        
지도 모드 변경(일반지도, 위성지도) 및 실시간 교통지도 및 자전거 지도 표시 방법은 샘플 프로젝트를 참고해 주시기 바랍니다.  
![Image of General Map](http://developer.naver.com/wiki/attach/Tutorial_Andriod/Android_2_1a.png)  
a) 일반지도  

![Image of Satellite Map](http://developer.naver.com/wiki/attach/Tutorial_Andriod/Android_2_1b.png)  
b) 위성지도

지도 뷰어 내부적으로 '네이버 지도앱 실행'버튼을 고정 노출할 수 있으며, 선택시 네이버 지도앱이 실행 됩니다.

'네이버 지도앱 실행' 버튼을 노출하려면 NNMapView 객체 생성 후 아래와 같이 설정 합니다.
```java
// enable built in app control; 
mMapView.setBuiltInAppControl(true);
```
앱에서 커스텀으로 버튼을 노출하고 선택시 '네이버 지도앱'을 실행하려면 아래와 같이 호출 합니다.
```java
mMapView.executeNaverMap();
```

<div id='id-section2'/>
## 지도 위에 오버레이 아이템 표시
이 예제에서는 지도 위에 오버레이 아이템을 표시하는 방법을 설명합니다.

__1) NMapOverlayManager 객체 생성__  
지도 위에 표시되는 오버레이 객체들을 관리하는 클래스입니다. 
각 오버레이 객체는 추가된 순서대로 화면에 렌더링되지만 이벤트 처리는 역순으로 마지막으로 추가된 오버레이 객체부터 처리됩니다. 
본 객체 생성 시에는 화면에 표시될 오버레이 객체에 대한 이미지 리소스를 제공하기 위하여 
아래와 같이 NMapResourceProvider 클래스를 상속받은 객체를 전달합니다.
```java
// create resource provider
mMapViewerResourceProvider = new NMapViewerResourceProvider(this);

// create overlay manager
mOverlayManager = new NMapOverlayManager(this, mMapView, mMapViewerResourceProvider);
```
NMapResourceProvider 클래스를 상속받은 객체 구현 방법은 샘플 프로젝트를 참고해 주시기 바랍니다.

__2) NMapPOIdataOverlay 객체 생성__   
여러 개의 오버레이 아이템을 하나의 오버레이 객체에서 관리하기 위한 오버레이 클래스이며 사용법은 아래와 같습니다.
```java
int markerId = NMapPOIflagType.PIN;

// set POI data
NMapPOIdata poiData = new NMapPOIdata(2, mMapViewerResourceProvider);
poiData.beginPOIdata(2);
poiData.addPOIitem(127.0630205, 37.5091300, "Pizza 777-111", markerId, 0);
poiData.addPOIitem(127.061, 37.51, "Pizza 123-456", markerId, 0);
poiData.endPOIdata();

// create POI data overlay
NMapPOIdataOverlay poiDataOverlay = mOverlayManager.createPOIdataOverlay(poiData, null);
```
                                                        
해당 오버레이 객체에 포함된 전체 아이템이 화면에 표시되도록 지도 중심 및 축적 레벨을 변경하려면 아래와 같이 구현합니다.
```java
// show all POI data
poiDataOverlay.showAllPOIdata(0);
```
아이템의 선택 상태가 변경되거나 말풍선이 선택되는 경우를 처리하기 위하여 이벤트 리스너를 등록합니다.
```java
// set event listener to the overlay
poiDataOverlay.setOnStateChangeListener(onPOIdataStateChangeListener);
```
해당 이벤트 발생 시 호출되는 인터페이스는 아래와 같이 구현합니다.
```java
public void onCalloutClick(NMapPOIdataOverlay poiDataOverlay, NMapPOIitem item) {
    // [[TEMP]] handle a click event of the callout
    Toast.makeText(NMapViewer.this, "onCalloutClick: " + item.getTitle(), Toast.LENGTH_LONG).show();
}

public void onFocusChanged(NMapPOIdataOverlay poiDataOverlay, NMapPOIitem item) {
    if (item != null) {
        Log.i(LOG_TAG, "onFocusChanged: " + item.toString());
    } else {
        Log.i(LOG_TAG, "onFocusChanged: ");
    }
}
```

__3) NMapCalloutOverlay객체 생성__   
NMapCalloutOverlay 클래스는 오버레이 아이템을 선택했을 때 표시되는 말풍선 오버레이의 기반 클래스입니다.   
지도 라이브러리에서 제공하는 말풍선 오버레이 대신에 사용자 정의 말풍선 오버레이로 변경하려면 
상기 클래스를 상속받은 객체를 생성해야 합니다.   
그리고, 사용자 정의 말풍선 오버레이를 NMapOverlayManager에 전달하기 위하여 아래와 같이 이벤트 리스너를 등록합니다.
```java
// register callout overlay listener to customize it.
mOverlayManager.setOnCalloutOverlayListener(onCalloutOverlayListener);
```
이후에 오버레이 아이템을 선택하면 아래의 콜백 인터페이스가 호출됩니다.
```java
public NMapCalloutOverlay onCreateCalloutOverlay(NMapOverlay itemOverlay, NMapOverlayItem overlayItem, Rect itemBounds) {
    // set your callout overlay
    return new NMapCalloutBasicOverlay(itemOverlay, overlayItem, itemBounds);
}
```
겹친 오버레이 아이템에 대해서 직접 처리하려면 null을 반환합니다. 
사용자 정의 말풍선 오버레이의 자세한 구현 방법은 샘플 프로젝트를 참고해 주시기 바랍니다.  
![Image of Overlay](http://developer.naver.com/wiki/attach/Tutorial_Andriod/Android_2_2.png)

<div id='id-section3'/>
## 지도 위에 경로 그리기
이 예제에서는 지도 위에 경로를 그리는 방법을 설명합니다. 지도 위에 경로를 그리기 위해서는 NMapPathDataOverlay 객체를 사용하여 아래와 같이 구현합니다.
```java
// set path data points
NMapPathData pathData = new NMapPathData(9);

pathData.initPathData();
pathData.addPathPoint(127.108099, 37.366034, NMapPathLineStyle.TYPE_SOLID);
pathData.addPathPoint(127.108088, 37.366043, 0);
pathData.addPathPoint(127.108079, 37.365619, 0);
pathData.addPathPoint(127.107458, 37.365608, 0);
pathData.addPathPoint(127.107232, 37.365608, 0);
pathData.addPathPoint(127.106904, 37.365624, 0);
pathData.addPathPoint(127.105933, 37.365621, NMapPathLineStyle.TYPE_DASH);
pathData.addPathPoint(127.105929, 37.366378, 0);
pathData.addPathPoint(127.106279, 37.366380, 0);
pathData.endPathData();

NMapPathDataOverlay pathDataOverlay = mOverlayManager.createPathDataOverlay(pathData);
```
전체 경로가 화면에 표시되도록 지도 중심 및 축적 레벨을 변경하려면 아래와 같이 구현합니다.
```java
// show all path data
poiDataOverlay.showAllPathData(0);
```
단순한 경로 이외에도 폴리곤과 원을 추가적으로 지도 위에 표시할 수 있으며 자세한 구현 방법은 
샘플 프로젝트를 참고해 주시기 바랍니다.  
![Image of path](http://developer.naver.com/wiki/attach/Tutorial_Andriod/Android_2_3.png)

<div id='id-section4'/>
## 지도 위 오버레이 아이템 위치 이동
이 예제에서는 지도 위를 터치하거나 오버레이 아이템을 끌어다 놓아서 아이템의 위치를 변경하는 방법을 설명합니다. 오버레이 아이템이 지도 위에 고정되지 않고 위치 이동이 가능하도록 아래와 같이 속성을 설정합니다
```java
int marker1 = NMapPOIflagType.PIN;

// set POI data
NMapPOIdata poiData = new NMapPOIdata(1, mMapViewerResourceProvider);

poiData.beginPOIdata(1);
NMapPOIitem item = poiData.addPOIitem(null, "Touch & Drag to Move", marker1, 0);

// initialize location to the center of the map view.
item.setPoint(mMapController.getMapCenter());

// set floating mode
item.setFloatingMode(NMapPOIitem.FLOATING_TOUCH | NMapPOIitem.FLOATING_DRAG);

// show right button on callout
item.setRightButton(true);

poiData.endPOIdata();

// create POI data overlay
NMapPOIdataOverlay poiDataOverlay = mOverlayManager.createPOIdataOverlay(poiData, null);
```
그리고 오버레이 아이템의 위치 변경을 처리하기 위하여 아래와 같이 이벤트 리스너를 등록합니다.
```java
poiDataOverlay.setOnFloatingItemChangeListener(onPOIdataFloatingItemChangeListener);
```
이후에 오버레이 아이템의 위치가 변경되면 아래의 콜백 인터페이스가 호출됩니다.
```java
public void onPointChanged(NMapPOIdataOverlay poiDataOverlay, NMapPOIitem item) {

   NGeoPoint point = item.getPoint();
   
   Log.i(LOG_TAG, "onPointChanged: point=" + point.toString());
}
```
![Image of OverlayItem](http://developer.naver.com/wiki/attach/Tutorial_Andriod/Android_2_4.png)

<div id='id-section5'/>
## 현재 위치 표시 및 나침반에 의한 지도 회전
이 예제에서는 지도 라이브러리에서 제공하는 현재 위치 표시 기능과 나침반에 의한 지도 회전 기능의 사용법을 설명합니다. 현재 위치 표시 기능을 사용하기 위해서는 AndroidManifest 파일에 아래와 같이 접근 권한을 설정합니다.
```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
```
지도 위에 현재 위치 표시와 나침반에 의한 지도 회전 기능을 구현하려면 아래와 같이 NMapMyLocationOverlay객체를 생성합니다.
```java
// location manager
mMapLocationManager = new NMapLocationManager(this);
mMapLocationManager.setOnLocationChangeListener(onMyLocationChangeListener);

// compass manager
mMapCompassManager = new NMapCompassManager(this);

// create my location overlay
mMyLocationOverlay = mOverlayManager.createMyLocationOverlay(mMapLocationManager, mMapCompassManager);      
```
그리고, 지도 회전 기능이 정상적으로 작동하기 위해서는 아래와 같이 NMapView 객체와 부모 뷰의 크기가 동일해야 하며 부모 뷰의 크기를 변경할 때에는 NMapView 객체의 크기를 적당하게 조정해 주어야 합니다.
```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // create map view
    mMapView = new NMapView(this);

    // create parent view to rotate map view
    mMapContainerView = new MapContainerView(this);
    mMapContainerView.addView(mMapView);

    // set the activity content to the map view
    setContentView(mMapContainerView);
```
현재 위치 탐색과 나침반에 의한 지도 회전 기능에 대한 자세한 사용법은 샘플 프로젝트 를 참고해 주시기 바랍니다.  
![Image of CompasMap](http://developer.naver.com/wiki/attach/Tutorial_Andriod/Android_2_5.png)

<div id='id-section6'/>
## 좌표를 주소로 변환 (Reverse geo-coding)
이 예제에서는 특정 좌표에 해당하는 주소를 구하는 방법을 설명합니다. 지도 라이브러리에서 제공하는 좌표-주소 변환 기능을 사용하기 위해서는 아래와 같이 Activity의 onCreate() 메서드에서 서버 데이터 이벤트 리스너를 등록해야 합니다.
```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    …
    // set data provider listener
    super.setMapDataProviderListener(onDataProviderListener);
}
```
그리고 특정 지점의 좌표를 인자로 아래와 같이 NMapActivity의 좌표-주소 변환 메서드를 호출합니다.
```java
  findPlacemarkAtLocation(point.longitude, point.latitude);
```  
이후에 서버로부터 좌표에 해당하는 주소를 전달받으면 아래의 콜백 인터페이스가 호출됩니다.
```java
public void onReverseGeocoderResponse(NMapPlacemark placeMark, NMapError errInfo) {
    if (errInfo != null) {
        Log.e(LOG_TAG, "Failed to findPlacemarkAtLocation: error=" + errInfo.toString());
        return;
    }
    Log.i(LOG_TAG, "onReverseGeocoderResponse: placeMark=" + placeMark.toString());
}
```
