---
layout: post
title:  "안전모 프로젝트 리뷰(부가기능부분)"
date:   2018-05-15 23:00:00
author: Seongjae Moon
categories: Android
tags:   안드로이드 자전거앱 OpenWeatherMap Android Project CapstoneDesign 안전모 안전하게자전거타는모임 OpenSource
cover: /assets/uploads/ajm/mainlogo1.jpg
---

[지난 포스팅](https://seongjaemoon.github.io/android/2018/05/06/ajmWeather.html)에 이어 역시나 오랜만에 안전모 프로젝트 리뷰~ 요즈음 일 복이 터져서 블로그에 글을 올리고 관리하는 게 잘 안 된다.. 좋은 기회가 많아져서 그런 것이라는 작은 위안?을 삼아보며 이번 포스팅에선 안전모의 환경 설정 부분과 자칭 꿀 기능 들에 대해서 알아보자. 다른 많은 기등들과 마찬가지로 많은 오픈 소스가 적절한 요소? 에 활용되었다. 그렇다면, 바로 안전모의 환경 설정과 부가기능 들에 대해서 알아보자.

# 환경 설정이 필요한 기능 구분
우선 환경 설정이라는 것은 말 그대로 앱에서 동작하는 여러 기능들에 대해서 사용자에게 선택권을 주고 사용자가 선택한 설정에 따라 앱이 다르게 동작하게 하는 것을 뜻한다고 할 수 있다. 대표적인 요소로 글자 폰트를 예로 들 수 있다. 하지만, 역시나 안전모에선 지원하지 않는다.. 아무튼, 안전모에서 제공하는 환경 설정 목록에 대해서 알아보자.  


항목|설명
:-:|:-:
정지 시간 감지|주행 평균 속도를 계산할 때 일시 정지 버튼을 누른 동안은 계산에 포함 되지 않는다. 
속도 표현 방법|Km/h, Mi/h 두 가지 표현 방법에 대한 설정이다.
경로 저장|지도 API 포스팅에서 보았던 경로 저장 부분을 설정하는 부분이다.
음성 안내 서비스|음성 안내 포스팅에서 보았던 음성 안내 서비스에 대한 설정 부분이다.
서비스 종료|앱을 종료할 때 모든 Service 클래스 기능을 종료하는 설정이다.
GPS 수신 감도|GPS 수신 감도를 설정하는 부분이다.
몸무게 입력|칼로리 계산을 위해 몸무게를 입력하는 부분이다.


위에 내용과 같이 안전모에서는 여러 가지 기능들에 대해서 변경 또는 실행 등을 설정할 수 있다. 물론 모든 경우의 수를 따져본 것은 아니지만, 적어도 나의 테스트 단계에선 이상 없이 동작했다... 그렇다면, 바로 코드를 살펴보자.
## 레이아웃 구성
우선 아래 xml 코드는 설정 화면 Activity를 위해 필요한 레이아웃 코드이다. 크게 특별한 부분은 없지만 FrameLayout 부분이 사용된 것을 확인할 수 있다. 아래에서 레이아웃에 대해 더 살펴보겠지만 결과적으로, PreferenceFragment 클래스를 상속받는 클래스를 이용해 설정 화면을 구성할 것이기 때문에 FrameLayout으로 따로 정의한 것이다. 
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

<android.support.v7.widget.Toolbar
        style="@style/AppTheme.PopupOverlay"
        android:id="@+id/settingstoolbar"
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="?attr/colorPrimary"
        android:minHeight="@dimen/abc_action_bar_default_height_material"
        android:elevation="5dp">
</android.support.v7.widget.Toolbar>

    <FrameLayout
        android:id="@+id/content_frame"
        android:layout_below="@+id/settingstoolbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</RelativeLayout>
```
아래 xml 코드는 실제 설정 화면에 보일 설정들에 대한 내용으로 res 폴더 밑에 xml 폴더를 만들고 따로 정의한다. PrefenceScreen이라는 고유한 이름을 갖는 엘리먼트를 정의하고, PreferenceCategory 부분 아래에 여러 Preference를 정의한다. 여기선 토글 형식으로 설정을 표현하는 SwitchPreference, 리스트 형식으로 설정을 표현하는 ListPreference, 사용자가 직접 값을 입력할 수 있는 EditTextPreference가 사용되었다.
```xml
<?xml version="1.0" encoding="utf-8"?>
<PreferenceScreen xmlns:android="http://schemas.android.com/apk/res/android">
    <PreferenceCategory
        android:title="@string/settings">
        <SwitchPreference
        android:key="auto_average"
        android:title="@string/pref_auto_average"
        android:summary="@string/pref_auto_average_summary" />

        <SwitchPreference
        android:key="miles_per_hour"
        android:title="@string/pref_miles_per_hour"
        android:summary="@string/pref_miles_per_hour_summary" />

        <SwitchPreference
        android:key="route"
        android:title="@string/route_saver"
        android:summary="@string/autosaving"/>

        <SwitchPreference
            android:key="alert"
            android:title="@string/alert_title"
            android:summary="@string/alertsetting" />

        <SwitchPreference
            android:key="autoservice"
            android:title="@string/autocancle_title"
            android:summary="@string/autocancel"/>

        <ListPreference
            android:key="gps_level"
            android:title="@string/gps_level"
            android:entries="@array/gps_weight"
            android:entryValues="@array/gps_values"
            android:dialogIcon="@drawable/ic_my_location_black_48dp"
            android:defaultValue="@string/default_gps_value"/>

        <EditTextPreference
            android:key = "weight_value"
            android:inputType="number"
            android:title ="@string/weight_value"
            android:summary="@string/weight_val"
            android:dialogIcon="@drawable/ic_add_alert_black_48dp"
            android:maxLines="1"
            android:singleLine="true"
            android:selectAllOnFocus="true"/>
    </PreferenceCategory>
</PreferenceScreen>
```
위 코드로 레이아웃을 구성하고 액티비티를 실행하면 아래와 같은 화면으로 구성되서 표현되게 된다. 각각의 속성은 이름 자체가 직관적이므로 어렵지 않게 이해할 수 있다.
![설정 화면](/assets/uploads/ajm/setting1.jpeg)

단, 한 가지 위의 내용만으론 부족한 부분이 있다. ListPreference 부분은 리스트로 보여야 할 부분이므로 리스트를 구성하는 부분이 필요한데, 속성 중 entries 부분과 entryValues 부분이 바로 그것이라고 할 수 있다. 아래 xml 코드를 보면 알 수 있듯이, 임의로 명명한 gps_weight 부분은 실제로 사용자에게 보일 리스트 부분이고, gps_values는 자바 코드상에서 실제 사용되는 값이라고 할 수 있다. 이를 위해 각각의 string-array 엘리먼트를 strings.xml에 따로 구성해주어야 한다.  
```xml
<!-- ...중략 -->
<string-array name="gps_values">
        <item>high</item>
        <item>middle</item>
        <item>low</item>
        <item>default</item>
</string-array>
<string-array name="gps_weight">
        <item>높은 GPS 수신강도</item>
        <item>중간 GPS 수신강도</item>
        <item>낮은 GPS 수신강도</item>
        <item>기본 GPS 수신강도</item>
</string-array>
<!-- ...중략 -->
```
![설정 화면](/assets/uploads/ajm/setting2.jpeg)
## java 코드
이번엔 자바 코드를 살펴보자. 
```java
package app.cap.ajm.Activity;
// 각종 클래스 import
import android.content.SharedPreferences;
import android.os.Bundle;
import android.preference.EditTextPreference;
import android.preference.ListPreference;
import android.preference.Preference;
import android.preference.PreferenceFragment;
import android.preference.PreferenceGroup;
import android.preference.PreferenceManager;
import android.support.v7.app.AppCompatActivity;
import android.view.MenuItem;
import android.widget.Toast;

import java.util.Locale;

public class SettingActivity extends AppCompatActivity implements SharedPreferences.OnSharedPreferenceChangeListener {
    private SharedPreferences sharedPreferences;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // onCreate 단계에서 settings 레이아웃과 연결한다.
        setContentView(R.layout.activity_settings);
        // 이 친구를 통해 설정 값을 받아오고 관리할 수 있게 된다.
        sharedPreferences = PreferenceManager.getDefaultSharedPreferences(getApplicationContext());

        android.support.v7.widget.Toolbar toolbar = (android.support.v7.widget.Toolbar) findViewById(R.id.settingstoolbar);
        setSupportActionBar(toolbar);
        getSupportActionBar().setHomeButtonEnabled(true);
        getSupportActionBar().setDisplayHomeAsUpEnabled(true);
        // 아래 코드를 통해 값을 쓰고 적용한다.
        getFragmentManager()
                .beginTransaction()
                .replace(R.id.content_frame, new SettingsFragment())
                .commit();
    }
    @Override
    protected void onResume() {
        super.onResume();
        sharedPreferences.registerOnSharedPreferenceChangeListener(this);
    }
    @Override
    public void onPause() {
        super.onPause();
        sharedPreferences.unregisterOnSharedPreferenceChangeListener(this);
    }

    //...중략

    // PreferenceFragment 클래스를 상속받는 Nested 클래스를 정의한다.
    public static class SettingsFragment extends PreferenceFragment {
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            // 위에서 작성한 Preference xml 파일을 연결한다.
            addPreferencesFromResource(R.xml.pref_general);
            // 설정이 변경되었을 경우를 인지해서 값을 다르게 보여준다.
            for (int i = 0; i < getPreferenceScreen().getPreferenceCount(); ++i) {
                Preference preference = getPreferenceScreen().getPreference(i);
                if (preference instanceof PreferenceGroup) {
                    PreferenceGroup preferenceGroup = (PreferenceGroup) preference;
                    for (int j = 0; j < preferenceGroup.getPreferenceCount(); ++j) {
                        updatePreference(preferenceGroup.getPreference(j));
                    }
                } else {
                    updatePreference(preference);
                }
            }
            // 칼로리 계산을 위한 몸무게 설정에 따른 값을 summary 속성에 표현하기 위한 코드이다. setOnPreferenceChangeListener를 구현해서 값이 변경된 경우 summary에 표현한다.
            final EditTextPreference editPref = (EditTextPreference)findPreference(KEY_WEIGHT_VALUE);
            editPref.setOnPreferenceChangeListener(new Preference.OnPreferenceChangeListener() {
                @Override
                public boolean onPreferenceChange(Preference preference, Object newValue) {
                    String news = newValue.toString();
                    if (!news.equals("")) {
                        int to = Integer.parseInt(news);
                        if (to < 10 || to > 120){
                            Toast.makeText(getActivity(), getString(R.string.weight_val), Toast.LENGTH_SHORT).show();
                            return false;
                        }else{
                            editPref.setSummary(String.valueOf(to));
                            return true;
                        }
                    }
                    return true;
                }
            });
            // 위 코드와 마찬가지로 ListPreference에 값이 변경되었을 경우를 확인하여 summary 값을 변경하기 위한 코드이다. 
            // 아래 코드를 보면 switch ~ case 문이 장황하게 사용되었는데.. 이는 안전모의 다국어 지원 때문에 벌어진 불상사?이다.
            final ListPreference listPref = (ListPreference)findPreference(KEY_GPS_VALUE);
            listPref.setOnPreferenceChangeListener(new Preference.OnPreferenceChangeListener() {
                @Override
                public boolean onPreferenceChange(Preference preference, Object newValue) {
                    String value = newValue.toString();
                    if (!value.equals("")) {
                        if (Locale.getDefault().getLanguage().equals("ko")) {
                            switch (value) {
                                case "high":
                                    listPref.setSummary("높은 GPS 수신감도");
                                    break;
                                case "middle":
                                    listPref.setSummary("중간 GPS 수신감도");
                                    break;
                                case "low":
                                    listPref.setSummary("낮은 GPS 수신감도");
                                    break;
                                case "default":
                                    listPref.setSummary("기본 GPS 수신감도");
                                    break;
                            }
                        } else if (Locale.getDefault().getLanguage().equals("en")) {
                            switch (value) {
                                case "high":
                                    listPref.setSummary("High GPS sensitivity");
                                    break;
                                case "middle":
                                    listPref.setSummary("Middle GPS sensitivity");
                                    break;
                                case "low":
                                    listPref.setSummary("Low GPS sensitivity");
                                    break;
                                case "default":
                                    listPref.setSummary("Default GPS sensitivity");
                                    break;
                            }
                        } else if (Locale.getDefault().getLanguage().equals("zh")) {
                            switch (value) {
                                case "high":
                                    listPref.setSummary("高GPS灵敏度");
                                    break;
                                case "middle":
                                    listPref.setSummary("中GPS灵敏度");
                                    break;
                                case "low":
                                    listPref.setSummary("低GPS灵敏度");
                                    break;
                                case "default":
                                    listPref.setSummary("默认GPS灵敏度");
                                    break;
                            }
                        }else if (Locale.getDefault().getLanguage().equals("ja")){
                            switch (value) {
                                case "high":
                                    listPref.setSummary("高いGPS感度");
                                    break;
                                case "middle":
                                    listPref.setSummary("中程度のGPS感度");
                                    break;
                                case "low":
                                    listPref.setSummary("低いGPS感度");
                                    break;
                                case "default":
                                    listPref.setSummary("デフォルトのGPS感度");
                                    break;
                            }
                        }else{
                            listPref.setSummary(value);
                        }
                        return true;
                    }
                    return false;
                }
            });
        }

        private void updatePreference(Preference preference)  {
            if (preference == null) {
                return;
            }
            if (preference.getKey().equals("gps_level")) {
                ListPreference listPreference = (ListPreference) preference;
                listPreference.setSummary(listPreference.getEntry());
                return;
            }
            if (preference.getKey().equals("weight_value")) {
                EditTextPreference editTextPreference = (EditTextPreference) preference;
                editTextPreference.setSummary(editTextPreference.getText());
            }
        }
    }
}
```
## 설정 활용
이제 남은 건 위의 코드에 설정 변경사항을 다른 액티비티 혹은 서비스 클래스에서 알아차리고 다른 동작을 진행하도록 해야 한다. 아래에선 설정에 따른 서비스 클래스의 동작을 다르게 하는 것에 대해서 살펴보자.
```java
package app.cap.ajm.Service;

//...중략
import android.content.SharedPreferences;
import android.preference.PreferenceManager;

public class GPSService extends Service implements LocationListener, TextToSpeech.OnInitListener{
    // 우선 SharedPreferences 인스턴스를 선언한다.
    private SharedPreferences sharedPreferences;
    @Override
    public void onCreate() {
        // onCreate 단계에서 PreferenceManager 클래스의 getDefaultSharedPreferences 메서드를 호출하여 할당한다. 
        // 인자로 context를 넘겨줘야 하는데, getApplicationContext 메서드를 넣어주자. 
        sharedPreferences = PreferenceManager.getDefaultSharedPreferences(getApplicationContext());

        // SharedPreferences 클래스는 여러 타입의 추상 메서드를 제공하는데, 넘겨주는 인자는 "값", "기본 값" 형태이다.
        // 아래 코드는 gps_level이라는 설정 값이 현재 어떻게 설정되어 있는지 받아오고 기본 값은 default로 설정된다는 코드이다.
        String gpsValue = sharedPreferences.getString("gps_level", "default");

        // 위의 코드처럼 임시 변수에 할당해서 사용하거나, 아래 코드처럼 바로 조건문에 넣어서 사용하면 된다. 반환 값은 메서드 이름과 동일한 타입의 반환 값을 리턴하게 된다. 
        if (sharedPreferences.getBoolean("route", false)){
            try {
                trackDBhelper.trackDBlocationRunning(Utils.INSTANCE.getCurrentSec(),lastLat, lastLon);
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
}
```

## 기타 기능
다음으로 간단하지만 개발할 땐 간단하지 않았던? 기능이라고 하기 뭐한 기능들에 대해서 알아보자.
### 언어 설정
먼저 안전모의 언어 설정 부분에 대해서 살펴보면, 말 그대로 설정 > 일반 > 언어 및 입력방식 화면에서 기본 언어를 변경하였을 경우 영어, 중국어, 일본어로 화면에 나타나는 언어가 다르게 표현되는 것을 말한다. 사실 제대로 된 다국어 지원을 위해선 각각의 언어에 능통한 분들의 도움이 필요하다. 하지만, 필자는 ~~마이웨이?~~이기 때문에 갓 구글 번역기를 많이 이용했다고 할 수 있다... 
- 준비 

다국어 지원을 위해서 res 폴더 밑에 values-en, values-ja, values-zh 폴더를 준비한다.(다른 언어도 가능하다.) 각각의 폴더는 이름에서 알 수 있듯이, 기존의 values 폴더에 있는 한국어 strings.xml에 대응하는 영어, 일본어, 중국어 strings.xml 파일이 들어있다.
- 변환

각각의 values 폴더에 들어있는 strings.xml 파일의 string 엘리먼트를 하나하나 번역하며 xml 파일을 만들기에는 너무 시간이 오래 걸린다. 이를 해결하기 위해서 구글의 스프레드 시트의 기능을 이용하면 된다. 변환과 관련된 더 자세한 방법은 [여기](http://dunji.tistory.com/2)에 아주 잘 설명되어 있다.
- 코드

기본적으로 위 방법을 사용하기 위해선 당연하지만 문자열 데이터가 하드코딩되어 있으면 안 된다. xml 코드는 android:text="@string/value" 형식으로 작성되어 있어야 하며, 자바 코드도 getResources(). getString(R.string.value) 형식으로 코딩되어 있어야 한다.

### 화면 잠금
이 기능은 안전모의 특성상 평균 속도가 20km/h가 넘어갈 경우 화면 터치가 불가능하게 막는 기능이라고 할 수 있다. 화면이 잠겼다면 잠금을 해제할 수 있어야 한다. 그래서 한 짓?이 바로 화면이 한 번 잠기면 멈춰서 볼륨 업 버튼을 클릭하게 하는 것이었다... ~~엄청난 발상~~ 홈 버튼이라던지 다른 트리거를 이용해도 되겠지 생각했지만 아무리, 아무리 해봐도 작동하지 않아서 찾아보니 보안 등 여러가지 이유로 그 정도까지는 핸들링하지 못하게 막아놨다고 한다. ~~역시 내가 못한게 아니었어!~~ 아무튼, 코드를 간단히 살펴보면 아래와 같다.

```java
import android.view.KeyEvent;

//...중략
// 속도가 20km/h 이상이면서 화면 터치가 잠기지 않았을 경우라면 
if (20 <= location.getSpeed() * 3.6 && !ishold) {
    // 화면 터치가 불가능하게 막는다.
    getWindow().addFlags(WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE);
    holder.setImageDrawable(ContextCompat.getDrawable(this, R.drawable.ic_lock_black_24dp));
    ishold = true;
}
//...중략
@Override
public boolean dispatchKeyEvent(KeyEvent event) {
    // 조건에 따른 볼륨 업 버튼 클릭 이벤트 처리
    if (event.getAction() == KeyEvent.ACTION_DOWN){
            if(event.getKeyCode() == KeyEvent.KEYCODE_VOLUME_UP&&!ishold){
                // 화면 터치 불가
                getWindow().addFlags(WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE);
                holder.setImageDrawable(ContextCompat.getDrawable(this, R.drawable.ic_lock_black_24dp));
                ishold = true;
                Toast.makeText(getApplicationContext(),getString(R.string.run_lock),Toast.LENGTH_SHORT).show();
                return true;
            }else if(event.getKeyCode() == KeyEvent.KEYCODE_VOLUME_UP&&ishold){
                // 화면 터치 가능
                getWindow().clearFlags(WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE);
                holder.setImageDrawable(ContextCompat.getDrawable(this, R.drawable.ic_lock_open_black_24dp));
                ishold = false;
                Toast.makeText(getApplicationContext(),getString(R.string.un_lock),Toast.LENGTH_SHORT).show();
                return true;
            }
        }
        return super.dispatchKeyEvent(event);
}
    //...중략
```
### 경로 공유
안드로이드 스마트폰을 6년 넘게 사용하면서 쓸 땐 몰랐지만, 막상 여러 가지 기능들을 구현하려고 보니 신기하게? 다가오는 기능들이 있었다. 바로 나의 앱에서 다른 앱을 실행시키거나 다른 앱으로 정보를 전달하는 등의 작업을 하는 조금은 특별한? 태스크 관리 부분이었다. 아이폰도 가능한지 모르겠지만, 아무튼 저장된 데이터를 문자열 데이터로 메신저 앱 등의 다른 앱으로 전송하는 간단한 방법에 대해서 알아보자.

```java
    //Intent 객체의 생성자에 Intent.ACTION_SEND를 넘겨주고 객체를 생성한다.
    Intent intents = new Intent(Intent.ACTION_SEND);
    // 탑이은 text/plain
    intents.setType("text/plain");
    // 각각 제목, 데이터 등의 형식이다.
    intents.putExtra(Intent.EXTRA_SUBJECT, getString(R.string.share));
    intents.putExtra(Intent.EXTRA_TEXT, data);
    intents.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
    startActivity(Intent.createChooser(intents, getTitle()));
```
<br><br>

이번 포스팅을 끝으로 안전모 프로젝트 리뷰를 모두 마쳤다. 하지만, 여전히 코드 리팩토링과 디버깅 작업은 끝나지 않았다.. 이게 바로 무한루프의 길? 아무튼, 처음 생각보다 굉장히 오래 걸린 것 같다. 많은 것을 글로 담고 싶었고, 생각보다 잘 되지 않았지만 나름 글로 작성하면서 다시 한 번 공부하게 되는 좋은 경험이 되었다. 글로 무언가 작성한다는 것이 이렇게 어려운 작업인 줄 알았다면 시작을 하지 않.. 아무튼, 다음 포스팅에선 안전모 프로젝트 에필로그를 작성해보는 걸로~

* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :) 
* [SharedPreferences에 대한 안드로이드 API 문서](https://developer.android.com/reference/android/content/SharedPreferences)