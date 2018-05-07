---
layout: post
title:  "안전모 프로젝트 리뷰(날씨정보부분)"
date:   2018-05-06 20:00:00
author: Seongjae Moon
categories: Android
tags:   안드로이드 자전거앱 OpenWeatherMap OpenAPI 날씨 Android Project CapstoneDesign 안전모 안전하게자전거타는모임 OpenSource
cover: /assets/uploads/ajm/mainlogo1.jpg
---

[지난 포스팅](https://seongjaemoon.github.io/android/2018/04/19/ajmFallDetection.html)에 이어 오랜만에 안전모 프로젝트 리뷰~ 이번 포스팅에선 안전모의 날씨 정보 제공 기능에 대해서 정리해보자. 다른 많은 기등들과 마찬가지로 오픈 소스를 기본으로 활용하여 개발되었고, 결과적으로 오픈웨더맵 API를 이용하여 날씨 정보를 호출하게 된다. 그렇다면, 안전모 날씨 정보 제공 기능에 대해서 알아보자.

# 오픈웨더맵 API 
![오픈웨더맵](/assets/uploads/ajm/owm.png)
우선 API를 사용하기 위해서 [여기](https://openweathermap.org/)에서 API 키를 발급 받아야 한다. 오픈웨더맵 사이트에 접속하여 회원가입을 하고, API 탭을 눌러 원하는 날씨 API를 선택하면 된다. 과거 날씨 정보를 비롯한 여러가지 API를 발급 받을 수 있는데, 안전모에선 Current weather data API를 사용한다.

## 키 발급
![오픈웨더맵](/assets/uploads/ajm/ownKey.png)
키를 정상적으로 발급받으면 위와 같이 새로운 API 키가 발급 받아진 것을 확인할 수 있다. 이제, 코드를 작성하여 날씨 정보를 요청하면 된다!

## 날씨 API 사용 준비
우선, API키를 안스에서 등록하는 방법은 여러가지가 있는데 날씨 정보 호출을 위해 참고한 EasyWeather 오픈 소스 프로젝트엔 app 폴더 밑의 build.gradle 파일에 API 키를 작성하고 호출하는 방식을 취한다. 기존의 다음카카오 지도 API를 호출할 때 처럼 string.xml에 등록하고 호출하거나 자바 코드 상에 문자열로 바로 등록하고 사용해도 무리 없이 동작한다. 여기선 build.gradle 파일에 API 키를 작성하고 불러오는 방식을 이용해보자.

- Root build.gradle 설정

우선 Root build,gradle 파일에 메이븐 https://jitpack.io 저장소를 추가한다.
```
allprojects {
		repositories {
			...
		    maven { url 'https://jitpack.io' }
		}
	}
```
- build.gralde 의존성 추가

아래와 같이 EasyWeather 오픈 소스를 사용하기 위해 app 폴더 밑의 build.gradle 파일에 의존성 코드를 추가하고 Sync Project를 클릭하여 프로젝트를 사용할 준비를 한다.
```
dependencies {
            ...
	    implementation 'com.github.code-crusher:EasyWeather:v1.2'
	}
```
참고로 안전모에서 프로젝트 자체를 다운로드하여 import 하는 방식을 취하고 있다. 만약 이와 같은 방법으로 프로젝트를 추가하고 싶다면, 루트 폴더에 프로젝트를 import 하고 아래와 같이 코드를 작성하면 된다.  
```
dependencies {
            ...
	    implementation project(':libraryf')
	}

```
위와 같이 코드를 작성하고 Sync Project가 정상적으로 이루어졌다면, 아래와 같이 API 키를 build.gradle 폴더에 추가하여 build 레벨에서 추가하도록 한다.
```
buildTypes.each {
        it.buildConfigField 'String', 'OWM_API_KEY', "\"API_KEY\""
    }
```
## 날씨 정보 데이터 준비
오픈웨더맵에서 제공하는 날씨 정보는 영어를 기본으로 값을 표현하고 있다. 온습도, 날씨 정보에 따른 이미지 등은 문자열 데이터가 아니기 때문에 그대로 사용해도 되지만, 비, 구름 조금 등의 날씨 정보 데이터는 영어로 되어있으면 바로 사용하기에 까다로움이 있을 수 있다. 안전모에선 이를 처리하기 위해 영어로 반환 받은 JSON 파일의 날씨에 해당하는 정보를 key : 영어, value : 한글 방식으로 미리 excel 파일로 정리해두고 이를 [안전모 데이터베이스](https://seongjaemoon.github.io/android/2018/02/20/ajmDB.html) 부분에서 살펴보았던 jxl 라이브러리를 이용해 엑셀 데이터를 불러와 SQLite 로컬 DB에 저장해두고 값을 비교하는 방식을 이용한다.
![](/assets/uploads/ajm/excel.png)
위 사진은 영어로 반환 받은 날씨 데이터의 한글 번역본 excel 파일을 캡처한 화면이다. 아래 엑셀 파일을 다운로드하여 assets 폴더에 저장한다.
[엑셀 파일 다운로드](/assets/uploads/ajm/weathers.xls) 

## 레이아웃 구성
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:fab="http://schemas.android.com/apk/res-auto"
    xmlns:materialdesign="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:keepScreenOn="true">
    <!-- ...중략 -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/weather1"
        android:textSize="15sp"
        android:text="@string/wet"
        android:layout_below="@+id/separator6"
        android:layout_marginTop="16dp"
        android:layout_marginLeft="20dp"
        android:layout_marginStart="20dp"/>
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/weather2"
        android:textColor="@color/black"
        android:textSize="15sp"
        android:layout_below="@+id/separator6"
        android:layout_toRightOf="@+id/weather1"
        android:layout_marginTop="16dp"
        android:layout_marginLeft="20dp"
        android:layout_marginStart="20dp"
        />
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/weather3"
        android:textSize="15sp"
        android:text="@string/temp"
        android:layout_below="@+id/weather1"
        android:layout_marginTop="16dp"
        android:layout_marginLeft="20dp"
        android:layout_marginStart="20dp"/>
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/weather5"
        android:textColor="@color/black"
        android:textSize="15sp"
        android:layout_below="@+id/weather2"
        android:layout_toRightOf="@+id/weather3"
        android:layout_marginTop="16dp"
        android:layout_marginLeft="20dp"
        android:layout_marginStart="20dp"
        />
     <!-- ...중략 -->
</RelativeLayout>
```
위 코드는 안전모의 메인화면에 보여지는 온도, 습도 정보를 나타내기 위해 TextView를 구성한 activity_main.xml 코드이다. 위와 같이 코드를 작성하면 아래와 같이 레이아웃 하단 왼쪽에 온도, 습도 정보를 표현할 수 있게 된다.
![안전모 메인 날씨](/assets/uploads/ajm/viewWeather1.jpeg)
더 다양한 날씨 정보를 표현할 수 있는 UI와 관련된 레이아웃 코드는 아래 첨부한 EasyWeather Github 저장소를 참고하면 된다. 여기선 바로 자바 코드를 살펴보도록 하자. 
## java 코드
우선 자바 코드는 기존에 만들어 둔 엑셀 파일을 데이터베이스에 저장하는 코드와 API 호출을 통해 반환 받은 날씨 정보를 한글로 표현하는 두 가지 코드로 나뉘게 된다.
### Weather
먼저 항상 그래왔듯이, 날씨 정보를 표현하기 위한 모델 클래스를 선언하고 생성자와 Getter를 정의한다. 
```java
public class Weather {

    private String key;
    private String value;

    public Weather(String key, String value){
        this.key = key;
        this.value = value;
    }

    public String getKey(){
        return key;
    }

    public String getValue(){
        return value;
    }

}
```
다음으로 아래 코드는 엑셀 데이터를 자바 코드 상으로 불러오고 저장하기 위한 DBhelper 클래스 코드이다.
### DBHelper
```java
import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.SQLException;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;
import android.util.Log;

import java.util.ArrayList;
import java.util.List;

import app.cap.ajm.Model.Weather;

public class WeatherDBhelper {
    public static final String KEY_ROWID = "_id";
    public static final String KEY_FOR_VALUE = "key"; // key
    public static final String KEY_CONTENT_NAME = "value";  //value

    private static final String TAG = WeatherDBhelper.class.getSimpleName();

    private DatabaseHelper mDbHelper;
    private SQLiteDatabase mDb;

    private static final String DATABASE_NAME = "weathers" ;//weather DB
    private static final String DATABASE_TABLE = "weatherTable";
    private static final int DATABASE_VERSION = 1;
    private static final String DATABASE_CREATE =
            "CREATE TABLE "+ DATABASE_TABLE +" ("
                    +KEY_ROWID+" INTEGER PRIMARY KEY AUTOINCREMENT, "
                    +KEY_FOR_VALUE+" TEXT,"
                    +KEY_CONTENT_NAME+" TEXT"+");";

    private final Context ctx;
    private List<Weather> weathers;
    private static class DatabaseHelper extends SQLiteOpenHelper {

        DatabaseHelper(Context context) {
            super(context, DATABASE_NAME, null, DATABASE_VERSION);
        }

        //버전이 업데이트 되었을 경우 DB를 다시 만들어 준다.
        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
            Log.w(TAG, "Upgrading database from version " + oldVersion + " to " + newVersion
                    + ", which will destroy all old data");

            db.execSQL("DROP TABLE IF EXISTS lists");
            onCreate(db);
        }

        //최초 DB를 만들때 한번만 호출된다.
        @Override
        public void onCreate(SQLiteDatabase db) {
            Log.w(TAG, "~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~onCreate~~~~~~~~~~~~~~~~~~~~~~~~~");
            db.execSQL(DATABASE_CREATE);
        }
    }

    public WeatherDBhelper(Context ctx){
        this.ctx = ctx;
    }

    public WeatherDBhelper open() throws SQLException{
        mDbHelper = new DatabaseHelper(ctx);
        mDb = mDbHelper.getWritableDatabase();
        return this;
    }

    public void close(){
        mDbHelper.close();
    }

    public long createList(String key, String value) {
        ContentValues initialValues = new ContentValues();
        initialValues.put(KEY_FOR_VALUE, key);
        initialValues.put(KEY_CONTENT_NAME, value);
        return mDb.insert(DATABASE_TABLE, null, initialValues);
    }
    // 날씨 정보 반환
    public List<Weather> fetchForList(){
        weathers = new ArrayList<>();
        mDbHelper = new WeatherDBhelper.DatabaseHelper(ctx);
        mDb =  mDbHelper.getReadableDatabase();
        Cursor res = mDb.query(DATABASE_TABLE, new String[]{KEY_ROWID, KEY_FOR_VALUE, KEY_CONTENT_NAME}, null, null, null, null, null);
        res.moveToFirst();
        while (!res.isAfterLast()) {
            String key = res.getString(res.getColumnIndex(KEY_FOR_VALUE));
            String value = res.getString(res.getColumnIndex(KEY_CONTENT_NAME));
            weathers.add(new Weather(key, value));
            res.moveToNext();
        }
        if (res.getCount()==0){
            return null;
        }
        res.close();
        return weathers;
    }

    //모든 레코드 반환
    public Cursor fetchAllList() {
        return mDb.query(DATABASE_TABLE, new String[]{KEY_ROWID, KEY_FOR_VALUE, KEY_CONTENT_NAME}, null, null, null, null,null);
    }
}
```
### SplashActivity
아래 코드는 위 코드와 연결되는 코드로 앱을 처음 실행했을 때 나오는 Splash Activity 코드에서 DBhelper 객체를 생성하여 해당 객체로 로컬 DB의 저장된 엑셀 데이터를 찾고, 만약 존재하지 않다면 데이터를 저장하고 이미 존재한다면, 기존의 데이터를 사용하게 된다.
```java
//...중략
import app.cap.ajm.Helper.WeatherDBhelper;
import jxl.Sheet;
import jxl.Workbook;

public class SplashActivity extends AppCompatActivity {
private WeatherDBhelper weatherDBhelper;
//... 중략
protected void onCreate(Bundle savedInstanceState) {
        //... 중략
        this.weatherDBhelper = new WeatherDBhelper(this);
        weatherDBhelper.open();
        Cursor cursor = weatherDBhelper.fetchAllList();
        Log.w("How many Value?:", String.valueOf(cursor.getCount()));
        if (cursor.getCount() == 0)
            excelToWeather();
        weatherDBhelper.close();
        cursor.close();
    }
    //... 중략
    // 날씨 정보를 DB에 저장한다.
    private void excelToWeather(){
        Workbook workbook = null;
        try{
            InputStream is = getBaseContext().getResources().getAssets().open("weathers.xls");
            workbook = Workbook.getWorkbook(is);
            if (workbook != null) {
                Sheet sheet = workbook.getSheet(0);
                if (sheet != null) {
                    int nMaxColumn = sheet.getColumns();
                    int nRowStartIndex = 0;
                    int nRowEndIndex = sheet.getColumn(nMaxColumn - 1).length - 1;
                    int nColumnStartIndex = 0;
                    weatherDBhelper.open();
                    for (int nRow = nRowStartIndex + 1; nRow <= nRowEndIndex; nRow++) {
                        String key = sheet.getCell(nColumnStartIndex, nRow).getContents();
                        String value = sheet.getCell(nColumnStartIndex + 1, nRow).getContents();
                        weatherDBhelper.createList(key, value);
                    }
                    weatherDBhelper.close();
                } else {
                    Log.i(TAG, "Sheet is null!!");
                }
            }else{
                    Log.i(TAG, "Woorkbook is null!!!");
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            if (workbook != null) {
                workbook.close();
            }
        }
    }
//...중략    
}
```
- MainActivity.java

아래 코드는 기존의 MainActivity 코드 중 날씨 정보를 표현하기 위한 코드 부분만 작성한 것이다. loadWeather("지역") 형태로 메서드를 호출하면 날씨 정보를 요청하게 되고, 구현된 임시 객체에 success, failure 값에 따라 weather2, weather5 텍스트뷰에 값이 보여지게 된다.
```java
import butterknife.BindView;
import github.vatsal.easyweather.Helper.ForecastCallback;
import github.vatsal.easyweather.Helper.TempUnitConverter;
import github.vatsal.easyweather.Helper.WeatherCallback;
import github.vatsal.easyweather.WeatherMap;
import github.vatsal.easyweather.retrofit.models.ForecastResponseModel;
import github.vatsal.easyweather.retrofit.models.WeatherResponseModel;
import butterknife.ButterKnife;
import android.widget.TextView;
import static app.cap.ajm.BuildConfig.OWM_API_KEY; // API 키 사용을 위해 import

//...중략
public class MainActivity extends AppCompatActivity {

    public final String weather_id = BuildConfig.OWM_API_KEY; //API 키
    @BindView(R.id.weather2) TextView weather2; //습도 정보를 표시할 TextView
    @BindView(R.id.weather5) TextView weather5; //온도 정보를 표시팔 TextView
    //...중략
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);
        //...중략
    }
    @Override
    protected void onResume() {
        loadWeather("seoul");
    }
    //...중략
    private void loadWeather(String city) {
        WeatherMap weatherMap = new WeatherMap(this, weather_id);
        weatherMap.getCityWeather(city, new WeatherCallback() {
            @Override
            public void success(WeatherResponseModel response) {
                populateWeather(response);
            }

            @Override
            public void failure(String message) {
            }
        });
        weatherMap.getCityForecast(city, new ForecastCallback() {
            @Override
            public void success(ForecastResponseModel response) {
            }

            @Override
            public void failure(String message) {
            }
        });
    }

    private void populateWeather(WeatherResponseModel response) {
        weather2.setText(response.getMain().getHumidity()+ "%");
        weather5.setText(TempUnitConverter.convertToCelsius(response.getMain().getTemp()).intValue() + "°C");
    }
    //...중략
}
```
- WeatherActivity.java

다음은 날씨 정보만 따로 보기 위해 클릭한 경우에 나타나는 WeatherActivity 코드이다. 여기서 한글로 날씨 정보를 표현하기 위해 만들어두었던, DBhelper 클래스 객체를 생성해서 값을 보여주게 된다. 위 코드와 많은 부분이 유사하니, 다른 부분만 코드로 표현했다.
```java
//...중략
import app.cap.ajm.Helper.WeatherDBhelper;
import static app.cap.ajm.BuildConfig.OWM_API_KEY;

public class WeatherActivity extends AppCompatActivity {
    private final String APP_ID = OWM_API_KEY;
    private final String city = "Seoul";
    @BindView(R.id.weather_title) TextView weatherTitle; // 한글로 표시하길 원하는 데이터
    //...중략
    private WeatherDBhelper weatherDBhelper;
    private List<app.cap.ajm.Model.Weather>weathers;
    //...중략
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_weather);
        ButterKnife.bind(this);
        loadWeather(city);
        weatherDBhelper = new WeatherDBhelper(this); //DBhelper 객체 생성
        weathers = new ArrayList<app.cap.ajm.Model.Weather>(); //한글로 날씨를 표시하기 위한 임시 List 컬렉션 객체 생성 (import한 패키지와 이름 충돌이 발생하므로 패키지명을 모두 작성한다.)
    }

    private void loadWeather(final String city) {
        WeatherMap weatherMap = new WeatherMap(this, APP_ID);
        weatherMap.getCityWeather(city, new WeatherCallback() {
            @Override
            public void success(WeatherResponseModel response) {
                populateWeather(response);
            }
            @Override
            public void failure(String message) {}
        });

        weatherMap.getCityForecast(city, new ForecastCallback() {
            @Override
            public void success(ForecastResponseModel response) {}
            @Override
            public void failure(String message) {
                Toast.makeText(getApplicationContext(), message, Toast.LENGTH_SHORT).show();
            }
        });
    }
    private void populateWeather(WeatherResponseModel response) {
        Weather weather[] = response.getWeather();
        condition.setText(weather[0].getMain());
       
        if(response.getName().equals("Seoul")&& Locale.getDefault().getLanguage().equals("ko")){
           location.setText("서울");
        }else{
            location.setText(response.getName());
        }
        String key = weather[0].getMain();
        // 언어 설정이 한글일 경우에 한해서 실행한다. (안전모는 다국어 지원을 위한 언어 설정이 따로 들어가있다.)
        if (Locale.getDefault().getLanguage().equals("ko")) {
            String value = getWeatherValue(key);
            //null 값 처리
            if(value !=null){
                condition.setText(value);
            }else{
                condition.setText(key);
            }
        }else {
            condition.setText(key);
        }
        String link = weather[0].getIconLink();
        Picasso.with(this).load(link).into(weatherIcon);
    }

    public String getWeatherValue(String key){
        String value = null;
        weatherDBhelper.open();
        // DB에서 모든 key를 받아온다.
        weathers = weatherDBhelper.fetchForList();
        for(app.cap.ajm.Model.Weather w : weathers){
            // 키(영어 날씨)에 해당하는 값을 할당한다.
            if(w.getKey().equals(key))
                value = w.getValue();
        }
        // 한글로 작성된 날씨 반환.
        return value;
    }

    //...중략
}
```
![안전모 메인 날씨](/assets/uploads/ajm/viewWeather2.jpeg)
위와 같이 날씨 정보에 "연무"라고 한글로 잘 표시되는 것을 확인 할 수 있다. 검색란의 Seoul이 아닌 다른 지역을 입력 하면(영어로만 가능) 다른 지역 날씨가 나타나게 된다. (ex. Busan) 아울러, 영어로만 가능한 부분은 코드를 통해 또 분기가 가능할 것으로 보인다.
<br><br>
이렇게 안전모 날씨 정보 부분에 대해서 간단하게 정리해보았다. 사실 EasyWeather 오픈 소스를 활용해서 작성된 코드가 많은 만큼, 직접 작성할 코드가 별로 없다는 장점?이 있다. 한글이 지원되지 않는 단점 때문에 약간의 코드를 가미하여 날씨 정보에 대한 한글 처리를 가능케 했다. 이런게 오픈 소스에 매력이 아닐까 싶다. ~~갑자기?~~ 아무튼, 다음은 안전모의 부가 기능 들에 대해서 알아보는 걸로~~

* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :) 
- [날씨 앱 참고 Github](https://github.com/code-crusher/EasyWeather).