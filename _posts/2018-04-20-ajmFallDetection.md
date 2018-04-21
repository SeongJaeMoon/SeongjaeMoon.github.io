---
layout: post
title:  "안전모 프로젝트 리뷰(넘어짐감지부분)"
date:   2018-04-19 23:00:00
author: Seongjae Moon
categories: Android
tags:   안드로이드 자전거앱 GPS 넘어짐감지 Android Project CapstoneDesign 안전모 안전하게자전거타는모임 OpenSource
cover: /assets/uploads/ajm/mainlogo1.jpg
---

[지난 포스팅](https://seongjaemoon.github.io/android/2018/04/06/ajmMap.html)에 이어 계속해서 안전모 프로젝트 리뷰~ 이번 포스팅에선 안전모의 넘어짐 감지 부분에 대해서 정리해보자. 안전모의 꽃?이자 가장 어려운 부분이라고 할 수 있다. 코드는 기존의 오픈 소스 코드를 최대한 활용하여 개발되었고, 개발을 위해 수학적 지식?이 많이 요구되는 부분이다. 그렇다면, 안전모 넘어짐 감지에 대해서 알아보자.

# Work Flow.
우선, 안전모에서 넘어짐 감지를 위해 자전거에 스마트 거치대를 이용하여 스마트폰을 거치하고 주행 중이라고 가정한다. 넘어짐 감지 액션에 대한 플로우는 아래와 같다.
1. 넘어짐 감지 서비스를 실행하기 전에, 넘어짐 감지시 위치를 전송할 연락처를 등록한다.
2. 넘어짐 감지 서비스가 실행되면 지속적으로 스마트폰 내장 센서를 이용한 회전 변화 감지한다.
3. 센서를 통해 갑작스러운 회전 변화를 감지하면, 사용자에게 응답을 요구하는 화면이 새롭게 보여진다.
4. 일정 시간 동안 응답이 없을 경우 이용자가 등록한 번호로 SMS를 이용해 위치를 자동 전송한다.

위의 내용과 같이 넘어짐 감지를 위해 연락처를 선 등록하고, 사용자가 넘어질 경우 자연스럽게 스마트폰의 회전 변화가 일어날 것이기 때문에, 이를 이용해 넘어짐 감지를 한다고 할 수 있다. 그렇다면, 플로우 순서대로 안전모의 넘어짐 감지에 대해 알아보자.
# 연락처 등록.
먼저, 연락처를 선 등록하는 Activity와 레이아웃 xml 구성, 그리고 등록된 연락처를 SQLite3에 저장할 DBhelper 부분이 필요하다. 하나하나 코드를 정리해보자.
## SMSDBhelper 클래스.
아래 코드는 사용자가 등록한 연락처 정보를 저장하기 위한 SQLite DB 관련 코드이다. [저번 포스팅](https://seongjaemoon.github.io/android/2018/04/06/ajmMap.html)에서 봤던 DB 관련 경로 저장 부분과 거의 유사한 액션으로 진행 되는 것을 알 수 있다.
```java
import android.database.*;
import android.database.sqlite.*;
import android.content.*;
//...중략

//SMSDBhelper 클래스
public class SMSDBhelper {
  private SQLiteDatabase mDb;
  private DatabaseHelper mDbHelper;

  private static final String DATABASE_NAME = "smslist.db";
  private static final int DATABASE_VERSION = 1;
  public static final String TABLE_NAME = "ContactList";
  public static final String COLUMN_CONTACT  = "contact";
  public static final String _ID = "id";
  private final Context mCtx;

    private static final String DATABASE_CREATE =
    "CREATE TABLE " +
    TABLE_NAME + "("+
    _ID + " INTEGER PRIMARY KEY AUTOINCREMENT" +","+
    COLUMN_CONTACT + " TEXT NOT NULL" + ");";

    public SMSDBhelper(Context context){
               this.mCtx =context;
    }

    private static class DatabaseHelper extends SQLiteOpenHelper {

        DatabaseHelper(Context context) {
            super(context, DATABASE_NAME, null, DATABASE_VERSION);
        }
        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        }
        @Override
        public void onCreate(SQLiteDatabase db) {
            db.execSQL(DATABASE_CREATE);
        }
    }

    public SMSDBhelper open() throws SQLException {
        mDbHelper = new SMSDBhelper.DatabaseHelper(mCtx);
        mDb = mDbHelper.getWritableDatabase();
        return this;
    }
    public void close() {
        mDbHelper.close();
    }
    public void addNewContact(String contact){
        ContentValues cv = new ContentValues();
        cv.put(COLUMN_CONTACT,contact);
        mDb.insert(TABLE_NAME,null,cv);
    }
    //SQLite는 기존의 RDBMS에서 사용하는 raw query와 자체적인 메서드를 이용한 쿼리 두 가지를 모두 지원한다.
    //아래 코드는 WHERE 조건절이 있는 delete 액션 코드와 저장된 전체 정보를 가져오는 select 쿼리 코드이다.
    public void removeContact(String contact){
        mDb.delete(TABLE_NAME, "contact"+"=?",new String[]{contact});
    }
    public Cursor getAllContacts(){
        return mDb.query(TABLE_NAME,null,null,null,null,null,COLUMN_CONTACT);
    }
}
```
## AccActivity 클래스.
위 SMSDBhelper 클래스를 통해 연착처 정보를 데이터베이스에 CRUD 작업 할 구성이 완료되었으니, 이제 이 DB를 사용할 AccActivity 코드를 아래와 같이 구현한다.
```java
import butterknife.ButterKnife;
//...중략

public class AccActivity extends AppCompatActivity{
  //각종 위젯을 구성한다.
  @BindView(R.id.add) Button addContacts;
  @BindView(R.id.start) Button start;
  @BindView(R.id.stop) Button stop;
  @BindView(R.id.removeAll) Button removeAll;
  @BindView(R.id.contacts) ListView lv; //연락처 목록을 보여줄 리스트뷰.
  @BindView(R.id.editText) EditText edit;

  private List<String> list = new ArrayList<>(); //adapter로 연결할 List 선언.
  private SMSDBhelper smsdBhelper; //연락처를 등록하기 위한 인스턴스 변수 선언.

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_acc);

        ButterKnife.bind(this);

        smsdBhelper = new SMSDBhelper(this);
        smsdBhelper.open();
        //모든 연락처 정보를 DB에서 가지고 온다.
        final Cursor cursor = smsdBhelper.getAllContacts();
        Toast.makeText(getApplicationContext(), getString(R.string.sms_alert), Toast.LENGTH_LONG).show();

        if (cursor.moveToFirst()) {
            do {
              //연락처 정보를 list에 저장한다.
                String data = cursor.getString(cursor.getColumnIndex(SMSDBhelper.COLUMN_CONTACT));
                list.add(data);
            } while (cursor.moveToNext());
        }
        //arrayAdapter 클래스를 이용해 저장된 값을 리스트뷰 형식으로 만든다.
        ArrayAdapter<String> arrayAdapter = new ArrayAdapter<>(this, R.layout.simplerow);
        arrayAdapter.addAll(list);
        lv.setAdapter(arrayAdapter);
    }
    //SensorService 시작 메서드.
    @OnClick(R.id.start) void onClickStartService(){
        Cursor mcursor = smsdBhelper.getAllContacts();
        mcursor.moveToFirst();
        int icount = mcursor.getInt(0);
        //등록된 번호가 하나라도 있을 경우라면
        if(icount > 0) {  
        //서비스 시작.
            Toast.makeText(getApplicationContext(),getString(R.string.start_service_safe),Toast.LENGTH_SHORT).show();
            Intent intent= new Intent(getApplicationContext(), SensorService.class);
            startService(intent);
        }else {
          //연락처 등록 요청 토스트 띄우기.
            Toast.makeText(getApplicationContext(),getString(R.string.at_least_number),Toast.LENGTH_SHORT).show();
        }
        mcursor.close();

      }    
    //...중략
}
```
## 레이아웃 구성.
AccActivity의 레이아웃은 연락처 입력창과 서비스 시작 버튼, 종료 버튼, 등록된 연락처 리스트를 나열할 리스트 뷰로 간단하게 구성되어 있다.
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_sms"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="app.cap.ajm.Activity.AccActivity">
    <TextView
        android:text="@string/set_number"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="15sp"
        />

    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:inputType="number"
        android:ems="10"
        android:id="@+id/editText" />
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/number_ok"
        android:textColor="@color/white"
        android:textStyle="bold"
        android:id="@+id/add"
        android:layout_marginTop="10dp"
        android:background="@color/red"/>

    <Button
        android:text="@string/start_detect"
        android:textColor="@color/white"
        android:textStyle="bold"
        android:background="@color/red_dark"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/start"
        android:layout_marginTop="10dp"
        />
    <!--...중략-->
    <ListView
        android:id="@+id/contacts"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_below="@+id/editText"
        android:layout_alignLeft="@+id/textView2"
        android:layout_alignStart="@+id/textView2"
        android:layout_marginTop="10dp">
    </ListView>
</LinearLayout>
```

<p style="text-align:center; font-size:20px;"> 위와 같이 레이아웃과 Activity를 구성하면 아래와 같은 화면이 구성되게 된다.</p>

![AccActivity](/assets/uploads/ajm/AccActivity.jpeg)
# 안드로이드 내장 센서.
위와 같이 연락처를 저장하고, 저장된 연락처로 SMS을 이용한 위치 정보를 전송할 준비를 완료했다면, 이제 실제로 넘어짐 감지를 위한 내장 센서를 활용해야 한다. 우선, 안드로이드 스마트폰엔(ios 기종도 마찬가지로) 여러 가지 내장 센서가 탑재되어 있다. 대표적으로 GPS 센서, 압력 센서, 온도 센서, 모션 센서, 기타 등등... 굉장히 많은 센서가 탑재되어 있다. 그중에 모션 센서 부분이 안전모에서 넘어짐 감지를 위해 사용되는 부분이라고 할 수 있다.

먼저, 안드로이드 모션 센서의 종류로는 자이로스코프 센서, 가속도계 센서, 지자기 센서가 있는데, 세 가지 센서의 측정값을 적절하게 조합해서 사용해야 한다. 자세한 내용은 아래서 하나하나 살펴보자.

# Roll, Pitch, Yaw.
우선 스마트폰의 회전 변화를 감지하기 위해서 필수적으로 알아야 하는 롤링, 피칭, 요잉이라는 것에 대해 알이보자.
![motionSensor](/assets/uploads/ajm/accelerometer.png)

결과적으로 위 사진과 같이 스마트폰은 오일러 각과 쿼터니언 회전 값에 대해 똑같이 적용되게 된다. 이 회전 변화 값은 앞서 언급한 세 가지 모션 센서를 활용하여 계산되게 되며, Service 클래스를 이용해 지속적으로 센서 값을 측정하여 갑작스러운 센서 값의 변화를 감지하는 것이 목표라고 할 수 있다.

원래 오일러 각과 회전 행렬 및 쿼터니언 회전 값은 움직이는 모든 강체에 적용되는 3차원 공간 좌표계이다. 인공위성의 PDOP, 비행 물체의 양력, 심지어 사람의 모션 인식에 이르기까지 웬만한 움직이고 회전하는 것에는 모두 적용될 수 있는 엄청난 계산법이다.. 단, 가속도계 센서나 자이로 센서를 바로 사용하기에는 오차가 누적되는 현상이 발생하기 때문에 이를 적절하게 소거하는 필터링 작업이 필요하다. 그렇다면, 오일러 각과 쿼터니언 회전에 대해서 정리해보자.

# Euler Angles and Rotation Matrix.
먼저, 오일러 각과 회전 행렬에 대한 개념을 알아보자.

회전행렬은 방향코사인행렬이라 부르기도 한다. 회전좌표계의 각 축이 기준좌표계의 각 축과 이루는 방향코사인으로 구성되기 때문이다.

$$ R = \begin{bmatrix}
n_x & o_x & a_x\\
n_y & o_y & a_y\\
n_z & o_z & a_z\\
\end{bmatrix} $$

회전행렬의 각 열벡터 $$ n = (n_x, n_y, n_z)^T, o = (o_x, o_y, o_Z)^T, z = (a_x, a_y, a_z)^T  $$는 단위벡터이고 서로 직교하는 성질이 있다.



$$ n \cdot o = 0, o \cdot a = 0, a \cdot n  = 0. $$

$$ \parallel n \parallel = 1, \parallel o \parallel = 1, \parallel a \parallel = 1.$$

$$ \therefore 회전행렬은 직교행렬(Orthogonal matrix) 이 되며, R^TR = I, R^T = R^{-1}를 만족한다.$$


다음으로, 오일러각(Euler Angles)는 3차원 공간에서 물체의 방위를 표시하기 위한 3개 각도의 조합이다. 오일러각으로 물체를 회전할 때는 회전 순서에 주의하여야 한다. 회전순서(X축->Y축->Z축 or Z축->Y축->X축) 등 각 축별 회전 순서의 조합)에 따라 물체의 회전된 최종 방위가 달라지기 때문이다.

일반적으로 ZYX 회전과 XYZ 회전이 주로 사용된다. 하지만 안전모는 여기서 YXZ(roll, pitch, azimuth) 회전을 사용하기 때문에 YXZ 수선의 따른 회전에 대해서 알아보자.

기준좌표계에 대한 물체좌표계의 회전 결과는 다음과 같이 계산된다.
1. 기준좌표계를 기준으로 Y-축을 중심으로 $$ \theta $$만큼 회전한다. $$ R_y(\theta) $$

2. 기준좌표계를 기준으로 X-축을 중심으로 $$ \phi $$만큼 회전한다. $$ R_x(\phi) $$

3. 기준좌표계를 기준으로 Z-축을 중심으로 $$ \psi $$만큼 회전한다. $$ R_z(\psi) $$

$$ R_x(\phi) = \begin{bmatrix} 1 & 0 & 0\\ 0 & cos\phi & sin\phi\\ 0 & -sin\phi & cos\phi\\ \end{bmatrix}, R_y(\theta) = \begin{bmatrix} cos\theta & 0 & sin\theta\\ 0 & 1 & 0\\ -sin\theta & 0 & cos\theta\\ \end{bmatrix}, R_z(\psi) = \begin{bmatrix} cos\psi & sin\psi & 0\\ -sin\psi & cos\psi & 0\\ 0 & 0 & 1\\ \end{bmatrix} $$

$$ R_{yxz} = \begin{bmatrix} cos\psi & sin\psi & 0\\ -sin\psi & cos\psi & 0\\ 0 & 0 & 1\\ \end{bmatrix} \cdot \begin{bmatrix} cos\theta & 0 & sin\theta\\ cos\phi sin\theta & sin\theta & cos\phi cos\theta\\ -cos\phi sin\theta & -sin\phi & cos\phi cos\theta\\ \end{bmatrix}$$

$$ \therefore R_{yxz} = \begin{bmatrix} cos\psi cos\theta + cos\phi cos\theta sin\psi & sin\psi sin\theta & cos\psi sin\theta + sin\psi cos\phi cos\theta \\ -sin\psi cos\theta + cos\psi cos\phi cos\theta & cos\psi sin\theta & -sin\psi sin\theta + cos\psi cos\phi cos\theta \\ -cos\phi sin\theta & -sin\phi & cos\phi cos\theta \\ \end{bmatrix} $$

위의 회전 변화를 간단하게 말로 표현하자면 아래와 같이 정리할 수 있다.
- Roll : 갸우뚱갸우뚱
- Pitch : 끄덕끄덕
- Yaw : 도리도리

# Angular Velocity.
위에서 계산을 끝내고 값을 구한 뒤, 가속계 센서를 이용해 각속도 값을 시간에 대해 적분한 값을 통해 최종 회전 변화 값을 구해낼 수 있다. 그렇다면, 각속도를 계산하기 위해 물체의 각속도와 회전 행렬의 관계에 대해 알아보자.

물체의 각속도는 물체가 단위시간당 회전한 양을 말한다. $$ \omega = (\omega_x, \omega_y, \omega_z)^T $$로 나타낼 수 있으며, 가속도 벡터 $$ \omega $$의 방향은 물체의 회전축을 나타내고, 벡터의 크기는 회전 속도를 나타낸다고 할 수 있다.

일단, 회전행렬 R은 다음과 같이 적분으로부터 계산할 수 있다.

$$ R = \int R\ dt = \int SR\ dt $$

하지만, 가속도계 센서는 그 특성상 갑작스러운 회전 변화에 대해서 오차가 누적되는 현상이 발생하게 된다. 때문에, 행렬의 직교성 $$ \tilde{R} \tilde{R} \neq I $$이 만족하지 않는다. 이를 정규화하기 위한 방법 중 하나로 아래와 같은 방법을 사용할 수 있다.

$$ R = \tilde{R}(\tilde{R}^T \tilde{R})^{1 \over 2} $$


# Quaternion.
쿼터니언이란 9개의 원소를 이용해 값을 계산하는 회전 행렬에 비해 4개의 원소만으로 같은 결과를 간결하게 표현할 수 있으며, 오일러 각 연산에서 발생하는 짐벌락을 해결할 수 있는 방안 중 하나이다. 여기서, 짐벌락이란 쉽게 말해 회전 축이 다른 회전 축에 종속적이므로, 내가 원하는 회전 축으로 계산되지 않는 것을 의미한다. 짐벌락에 대한 더 자세한 내용은 [여기](http://hoodymong.tistory.com/3)에 아주 잘 설명되어 있다.

쿼터니언은 3개의 벡터 요소와 하나의 스칼라 요소로 구성된다.

$$ q = \left\{\eta, \varepsilon \right\} = \left\{\eta, \varepsilon, \varepsilon, \varepsilon \right\} $$

쿼터니언은 크기가 1일 때 단위쿼터니언(Unit Quaternion)이라고 부르는데, 단위 쿼터니언은 쿼터니언의 놈을 나누워 계산한다.

$$ q_u = {q \over \parallel q \parallel} 여기서 놈은, 다음과 같이 정의된다. $$

$$ \parallel q \parallel = \sqrt{\eta^2 + \varepsilon^2 + \varepsilon^2 + \varepsilon^2} $$

시간의 변화에 따라 적분한 값의 따른 쿼터니언 변환은 다음과 같다.

$$ q = Q(\Lambda) = Q_{delta}(\theta)\\ = \begin{bmatrix} \eta \\ \varepsilon_x\\ \varepsilon_y\\ \varepsilon_z \end{bmatrix} = \begin{bmatrix} sin({\theta \over 2} \cdot \parallel q \parallel) \\ sin({\theta \over 2} \cdot \parallel q \parallel) \\ sin({\theta \over 2} \cdot \parallel q \parallel)\\ cos({\theta \over 2})\end{bmatrix} $$

계산의 최종 순서는 오일러 각 -> 쿼터니언 변환 -> 회전 행렬 계산이며, 이제 남은건 이 내용을 다시 코드로 작성하는 ~~헬...~~ 작업이라고 할 수 있다.

# 넘어짐 감지.
사실 위의 내용은 아래 코드의 전체 내용을 아우르진 않는다. 회전 행렬과 쿼터니언에 대한 기본적인 내용을 정리한 것이고, 실제 코드는 위 내용보다 약간 더 난이도 있는 내용이 필요하다.  
```java
import android.hardware.Sensor;
import android.hardware.SensorEvent;
import android.hardware.SensorEventListener;
import android.hardware.SensorManager;
import java.util.HashMap;
import java.util.Locale;
import java.util.Timer;
import java.util.TimerTask;
import android.os.Bundle;
import android.os.Handler;
import android.os.Looper;
//...중략
public class SensorService extends Service implements SensorEventListener{
    private double latitude, longitude; //GPS를 통해 측정된 위치 값을 다른 액티비티로 넘겨주기 위한 변수
    Handler handler = new Handler(Looper.getMainLooper()); // 메인 쓰레드
    private Handler mPeriodicEventHandler = new Handler(); // TIME OUT 쓰레드
    private final int PERIODIC_EVENT_TIMEOUT = 3000; // TIME OUT 세팅
    //서비스에서 작동될 쓰레드 시간 관리 타이머
    private Timer fuseTimer = new Timer();
    private int sendCount = 0;
    private char sentRecently = 'N';    
    //자이로 센서의 각속도
    private float[] gyro = new float[3];
    //계산된 각속도
    private float degreeFloat;
    private float degreeFloat2;
    //자이로 센서 데이터의 회전 행렬
    private float[] gyroMatrix = new float[9];
    //자이로 행렬으로부터의 방위각
    private float[] gyroOrientation = new float[3];
    //자기장 벡터
    private float[] magnet = new float[3];
    //가속도계 벡터
    private float[] accel = new float[3];
    //가속도계와 자기장으로부터의 방위각
    private float[] accMagOrientation = new float[3];
    //3가지 센서를 합친 것의 방위각
    private float[] fusedOrientation = new float[3];
    //가속도계와 자기장센서의 기준 회전 행렬
    private float[] rotationMatrix = new float[9];
    //센서 값의 변화 크기를 비교하기 위한 변수
    public static final float EPSILON = 0.000000001f;
    //타이머의 쓰레드의 간격 설정용 변수
    public static final int TIME_CONSTANT = 30;
    //중력 가속도값
    public static final float FILTER_COEFFICIENT = 0.98f;
    //나노s -> s
    private static final float NS2S = 1.0f / 1000000000.0f;
    //시간으로 적분하기 위한 변수
    private float timestamp;
    //재실행을 위한 변수
    private boolean initState = true;
    //안드로이드 센서를 사용하기 위한 변수
    private SensorManager senSensorManager;
    private Sensor senAccelerometer;
    private Sensor senProximity;
    private SensorEvent mSensorEvent;
    //메시지 전송 이벤트 처리를 위한 Runnable 임시 객체 구현
    private Runnable doPeriodicTask = new Runnable() {
            public void run() {
                sentRecently = 'N';
          }
    };
  @Nullable
   @Override
   public IBinder onBind(Intent intent) {
       return null;
   }

   @Override
   public void onCreate() {
       super.onCreate();
   }

   @Override
   public void onDestroy() {
       super.onDestroy();
       mPeriodicEventHandler.removeCallbacks(doPeriodicTask);
       senSensorManager.unregisterListener(this);
       sendCount = 0;
   }
   @Override
   public int onStartCommand(Intent intent, int flag, int startId) {
     //서비스가 시작되면 센서를 사용하기 위한 객체 할당.
     senSensorManager = (SensorManager) getSystemService(Context.SENSOR_SERVICE);
     senAccelerometer = senSensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER);     
     //간혹 낮은 버전의 안드로이드 기종은 가속도계 센서가 사용 불가한 경우가 있으니, 확인 작업을 한다.
     if(senSensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER)!=null) {
                 initListeners();
                 //타이머 시작.
                 fuseTimer.scheduleAtFixedRate(new calculateFusedOrientationTask(), 1000, TIME_CONSTANT);
             }else {
                 Toast.makeText(getApplicationContext(), getString(R.string.not_supprot_acc), Toast.LENGTH_SHORT).show();
             }

             return START_STICKY;
   }
   //센서 초기화.
   public void initListeners() {
       senSensorManager.registerListener(this,
               senSensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER),
               SensorManager.SENSOR_DELAY_FASTEST);

       senSensorManager.registerListener(this,
               senSensorManager.getDefaultSensor(Sensor.TYPE_GYROSCOPE),
               SensorManager.SENSOR_DELAY_FASTEST);

       senSensorManager.registerListener(this,
               senSensorManager.getDefaultSensor(Sensor.TYPE_MAGNETIC_FIELD),
               SensorManager.SENSOR_DELAY_FASTEST);
   }
   //SensorEventListener 인터페이스의 속한 메서드를 오버라이딩해서 구현 한다.
   @Override
   public void onSensorChanged(SensorEvent sensorEvent) {
        Sensor mySensor = sensorEvent.sensor;

        switch (sensorEvent.sensor.getType()) {
             case Sensor.TYPE_ACCELEROMETER:
                 // 새로운 가속도계 데이터를 가속도계 배열에 복사
                 // 새로운 방위각 계산
                 System.arraycopy(sensorEvent.values, 0, accel, 0, 3);
                 calculateAccMagOrientation();
                 break;
             case Sensor.TYPE_GYROSCOPE:
                 // 자이로 데이터 처리
                 gyroFunction(sensorEvent);
                 break;
             case Sensor.TYPE_MAGNETIC_FIELD:
                 //새로운 자기장 데이터를 배열에 복사
                 System.arraycopy(sensorEvent.values, 0, magnet, 0, 3);
                 break;
           }
       }
       @Override
       public void onAccuracyChanged(Sensor sensor, int i) {
       }

       public void calculateAccMagOrientation() {
        if(SensorManager.getRotationMatrix(rotationMatrix, null, accel, magnet)){
            SensorManager.getOrientation(rotationMatrix, accMagOrientation);
         }
       }

       private void getRotationVectorFromGyro(float[] gyroValues,float[] deltaRotationVector, float timeFactor) {
               float[] normValues = new float[3];

               //샘플의 각속도를 계산한다.
               float omegaMagnitude =
                       (float) Math.sqrt(gyroValues[0] * gyroValues[0] +
                               gyroValues[1] * gyroValues[1] +
                               gyroValues[2] * gyroValues[2]);

               //축을 얻기에 충분히 큰 경우, 회전 벡터를 표준화
               if (omegaMagnitude > EPSILON) {
                   normValues[0] = gyroValues[0] / omegaMagnitude;
                   normValues[1] = gyroValues[1] / omegaMagnitude;
                   normValues[2] = gyroValues[2] / omegaMagnitude;
               }


               /*timestep에 의해 이 축을 중심으로 각속도와 통합한다.
               이 샘플에서 시간 경과에 따른 델타 값의 회전변환을 얻으려면 델타 회전의 축각 표현의 변환이 필요하다.
               즉, 회전 행렬로 변환하기 전에 쿼터니언으로 변환 */
               float thetaOverTwo = omegaMagnitude * timeFactor;
               float sinThetaOverTwo = (float) Math.sin(thetaOverTwo);
               float cosThetaOverTwo = (float) Math.cos(thetaOverTwo);
               deltaRotationVector[0] = sinThetaOverTwo * normValues[0];
               deltaRotationVector[1] = sinThetaOverTwo * normValues[1];
               deltaRotationVector[2] = sinThetaOverTwo * normValues[2];
               deltaRotationVector[3] = cosThetaOverTwo;
           }
        private void gyroFunction(SensorEvent event) {
                 //첫 번째 가속도계 / 자기장 방향이 획득 될 때까지 시작하지 않음.
                  if (accMagOrientation == null)
                     return;

                 //자이로 회전 배열 값을 초기화한다.
                 if (initState) {
                     float[] initMatrix = new float[9];
                     initMatrix = getRotationMatrixFromOrientation(accMagOrientation);
                     float[] test = new float[3];
                     SensorManager.getOrientation(initMatrix, test);
                     gyroMatrix = matrixMultiplication(gyroMatrix, initMatrix);
                     initState = false;
                 }

                 //새 자이로 값을 자이로 배열에 복사한다.
                 //원래의 자이로 데이터를 회전 벡터로 변환한다.
                 float[] deltaVector = new float[4];
                 if (timestamp != 0) {
                     final float dT = (event.timestamp - timestamp) * NS2S;
                     System.arraycopy(event.values, 0, gyro, 0, 3);
                     getRotationVectorFromGyro(gyro, deltaVector, dT / 2.0f);
                 }

                 //측정 완료이 완료되면, 다음 시간 간격을 위해 현재 시간을 설정한다.
                 timestamp = event.timestamp;

                 //회전 벡터를 회전 행렬로 변환한다.
                 float[] deltaMatrix = new float[9];
                 SensorManager.getRotationMatrixFromVector(deltaMatrix, deltaVector);
                 //회전 벡터를 회전 행렬로 변환한다.
                 gyroMatrix = matrixMultiplication(gyroMatrix, deltaMatrix);
                 //회전 행렬에서 자이로 스코프 기반 방향을 얻는다.
                 SensorManager.getOrientation(gyroMatrix, gyroOrientation);
               }
        //회전행렬을 구하기 위한 메서드       
        private float[] getRotationMatrixFromOrientation(float[] o) {
                 float[] xM = new float[9];
                 float[] yM = new float[9];
                 float[] zM = new float[9];

                 float sinX = (float) Math.sin(o[1]);
                 float cosX = (float) Math.cos(o[1]);
                 float sinY = (float) Math.sin(o[2]);
                 float cosY = (float) Math.cos(o[2]);
                 float sinZ = (float) Math.sin(o[0]);
                 float cosZ = (float) Math.cos(o[0]);

                 //x 축 (피치)에 대한 회전배열
                 xM[0] = 1.0f;
                 xM[1] = 0.0f;
                 xM[2] = 0.0f;
                 xM[3] = 0.0f;
                 xM[4] = cosX;
                 xM[5] = sinX;
                 xM[6] = 0.0f;
                 xM[7] = -sinX;
                 xM[8] = cosX;

                 //y 축 (롤)에 대한 회전배열
                 yM[0] = cosY;
                 yM[1] = 0.0f;
                 yM[2] = sinY;
                 yM[3] = 0.0f;
                 yM[4] = 1.0f;
                 yM[5] = 0.0f;
                 yM[6] = -sinY;
                 yM[7] = 0.0f;
                 yM[8] = cosY;

                 //z 축에 대한 회전 (방위각)배열
                 zM[0] = cosZ;
                 zM[1] = sinZ;
                 zM[2] = 0.0f;
                 zM[3] = -sinZ;
                 zM[4] = cosZ;
                 zM[5] = 0.0f;
                 zM[6] = 0.0f;
                 zM[7] = 0.0f;
                 zM[8] = 1.0f;

                 //회전 순서는 y, x, z (롤, 피치, yaw)
                 float[] resultMatrix = matrixMultiplication(xM, yM);
                 resultMatrix = matrixMultiplication(zM, resultMatrix);

                 return resultMatrix;

               }
        //회전행렬의 곱을 계산하기 위한 메서드.
        private float[] matrixMultiplication(float[] A, float[] B) {
              float[] result = new float[9];
                result[0] = A[0] * B[0] + A[1] * B[3] + A[2] * B[6];
                result[1] = A[0] * B[1] + A[1] * B[4] + A[2] * B[7];
                result[2] = A[0] * B[2] + A[1] * B[5] + A[2] * B[8];

                result[3] = A[3] * B[0] + A[4] * B[3] + A[5] * B[6];
                result[4] = A[3] * B[1] + A[4] * B[4] + A[5] * B[7];
                result[5] = A[3] * B[2] + A[4] * B[5] + A[5] * B[8];

                result[6] = A[6] * B[0] + A[7] * B[3] + A[8] * B[6];
                result[7] = A[6] * B[1] + A[7] * B[4] + A[8] * B[7];
                result[8] = A[6] * B[2] + A[7] * B[5] + A[8] * B[8];

                return result;
            }
        //지속적으로 센서 값의 변화를 계산할 쓰레드 세팅.                   
        class calculateFusedOrientationTask extends TimerTask {
                 public void run() {
                     float oneMinusCoeff = 1.0f - FILTER_COEFFICIENT;
                     //세가지 센서의 최종 측정값을 필터링.
                     fusedOrientation[0] =
                             FILTER_COEFFICIENT * gyroOrientation[0]
                                     + oneMinusCoeff * accMagOrientation[0];

                     fusedOrientation[1] =
                             FILTER_COEFFICIENT * gyroOrientation[1]
                                     + oneMinusCoeff * accMagOrientation[1];

                     fusedOrientation[2] =
                             FILTER_COEFFICIENT * gyroOrientation[2]
                                     + oneMinusCoeff * accMagOrientation[2];

                     //**********가속도계 센서 값의 변화 측정**********
                     double SMV = Math.sqrt(accel[0] * accel[0] + accel[1] * accel[1] + accel[2] * accel[2]);
                      //여기선 35로 되어있지만 값이 25 정도만 되어도 충분히 회전 변화를 측정 가능하다.
                     if (SMV > 35 ) {
                         GPSService gpsService = new GPSService();
                         if (gpsService.getLastlocation() != null) {
                             if (sentRecently == 'N') {
                                 Log.d("Accelerometer vector:", "" + SMV);
                                 //라디안 값을 각도로 표현.
                                 degreeFloat = (float) (fusedOrientation[1] * 180 / Math.PI);
                                 degreeFloat2 = (float) (fusedOrientation[2] * 180 / Math.PI);
                                 if (degreeFloat < 0)
                                     degreeFloat = degreeFloat * -1;
                                 if (degreeFloat2 < 0)
                                     degreeFloat2 = degreeFloat2 * -1;
                                 if (degreeFloat > 30 || degreeFloat2 > 30) {
                                     Log.d("Degree1:", "" + degreeFloat);
                                     Log.d("Degree2:", "" + degreeFloat2);
                                     //이 곳에 넘어짐 감지시 필요헌 코드 작성.
                                     speak(getString(R.string.fall_detect));
                                     //새로운 화면을 띄우기 위한 액션 처리.
                                     Intent intent = new Intent(SensorService.this, DialogActivity.class);
                                     intent.putExtra("lastlat", gpsService.getLastlocation().getLatitude());
                                     intent.putExtra("lastlon", gpsService.getLastlocation().getLongitude());
                                     intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                                     startActivity(intent);
                                 } else {
                                     handler.post(new Runnable() {
                                         @Override
                                         public void run() {
                                           //센서값 변화는 측정. but, 회전 축이  x-z, y-z 간의 회전 변화가 아니므로 다른 액션을 발생한다.
                                             Toast.makeText(SensorService.this.getApplicationContext(), getString(R.string.be_careful), Toast.LENGTH_LONG).show();
                                             speak(getString(R.string.be_careful_tts));
                                             Log.d("Send!", "센서 값 변화!!!!! " + sendCount);
                                         }
                                     });
                                     sendCount++;
                                 }
                                 sentRecently = 'Y';
                                 //쓰레드를 지연시킨다.
                                 mPeriodicEventHandler.postDelayed(doPeriodicTask, PERIODIC_EVENT_TIMEOUT);
                             }
                         }
                     }
                     //측정 값을 다시 덮어쓴다.
                     gyroMatrix = getRotationMatrixFromOrientation(fusedOrientation);
                     System.arraycopy(fusedOrientation, 0, gyroOrientation, 0, 3);
                 }
             }            
}
```
# 응답 요구.
위의 코드를 통해 센서의 회전 변화를 측정하였다면, 다음으로 그에 따른 액션이 진행되어야 한다. 안전모에선 DialogActivity라는 것을 이용해 버튼 타입의 액티비티를 새로 띄워 사용자에게 응답을 요구한다.

## AndroidManifest.xml.
먼저 액티비티를 매니페스트에 등록할 때 theme 속성의 값을 @style/Base.Theme.AppCompat.Dialog로 설정한다. 이렇게 되면, 액티비가 Dialog 형식으로 작성될 수 있다.
```xml
<activity
  android:name=".Activity.DialogActivity"
  android:theme="@style/Base.Theme.AppCompat.Dialog" />
```
## 레이아웃 구성.
우선, 응답요구를 위한 액티비티는 버튼 형식의 다이얼로그를 띄울 것이기 때문에, 레이아웃에 아무것도 선언하지 않는다.
```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/white"
    tools:context="app.cap.ajm.Activity.DialogActivity">
</android.support.constraint.ConstraintLayout>
```
다음으로, custom_dialog라는 xml 파일을 만들고, 여기에 Button 위젯 하나만 선언한다. 보여질 메세지는 text 속성을 통해 설정한다.
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <Button
        android:id="@+id/dialogButtonOK"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:text="@string/are_u_ok"
        android:textSize="40sp"
        android:textColor="@color/red_dark"
        style="@style/Widget.AppCompat.Button.Colored"
        tools:ignore="RtlHardcoded,RtlSymmetry" />

</RelativeLayout>
```
## DialogActivity.java 클래스.
이제 아래 코드와 같이 자바 코드를 작성하면, 새로운 액티비티 전체 화면을 버튼 타입으로 처리할 수 있게 된다. 이 액티비티 화면은 서비스 클래스에서 실행시키는 것이기 때문에, 서비스만 실행되어 있다면 어떤 화면에서든 새로운 액티비티를 그리는 것이 아닌 버튼 타입의 액티비티만 위에 새롭게 생성되게 된다. 얄루!
```java
@Override
protected void onCreate(Bundle savedInstanceState) {

    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_dialog);

    handler = new Handler();
    final Dialog dialog = new Dialog(context);

    dialog.setContentView(R.layout.custom_dialog);
    Button dialogButton = (Button) dialog.findViewById(R.id.dialogButtonOK);
    dialogButton.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            handler.removeCallbacksAndMessages(null);
            dialog.dismiss();
            finish();

        }
    });
    dialog.show();
```
![넘어짐 감지](/assets/uploads/ajm/falling.jpeg)

<p style = "text-align:center;">디자인은 보지 말자..</p>

# SMS 위치 전송.
이제 마지막으로 남은 것은, 사용자의 응답이 업을 경우 SMS를 통해 SQLite에 저장된 연락처에 위치 정보를 전송하는 것이다. 먼저, 아래와 같이 권한을 획득할 수 있도록 설정한다. SMS 권한도 런타임 권한이므로, 기능을 실행할 때 권한을 따로 획득하는 작업이 필요하다.
```xml
<uses-permission android:name="android.permission.SEND_SMS" />
```
## DialogActivity.java
아래 코드는 위의 코드와 이어지는 내용으로, 약 7초간의 응답 시간 동안 응답이 없을 경우 SMS를 전송한다.  
```java
ackage app.cap.ajm.Activity;

import android.app.Dialog;
import android.content.Context;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Handler;
import android.os.Bundle;
import android.telephony.SmsManager;
import app.cap.ajm.Helper.SMSDBhelper;
//...중략

public class DialogActivity extends AppCompatActivity{

    private Handler handler;
    private final Context context = this;
    private String phoneNum = "";
    private String textMsg;
    private String prevNumber;
    private SQLiteDatabase sqls;
    private static final String TABLE_NAME = "ContactList";
    private static final String COLUMN_CONTACT  = "contact";

    @Override
    protected void onCreate(Bundle savedInstanceState) {

        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_dialog);
        final SMSDBhelper smsdBhelper = new SMSDBhelper(this);
        smsdBhelper.open();

        handler = new Handler();
        final Dialog dialog = new Dialog(context);

        dialog.setContentView(R.layout.custom_dialog);
        Button dialogButton = (Button) dialog.findViewById(R.id.dialogButtonOK);
        dialogButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                handler.removeCallbacksAndMessages(null);
                dialog.dismiss();
                finish();

            }
        });
        dialog.show();


        handler.postDelayed(new Runnable() {
            @Override
            public void run() {
                Intent intent = getIntent();
                try {
                  //넘겨받은 위치 정보.
                    double latitude = intent.getExtras().getDouble("lastlat");
                    double longitude = intent.getExtras().getDouble("lastlon");
                    List<String> itemIds = new ArrayList<>();
                    Cursor cursor = smsdBhelper.getAllContacts();
                    smsdBhelper.close();
                    cursor.moveToFirst();
                    //저장된 연락처만큼 전송한다.
                    if (cursor.moveToFirst()) {
                        do {
                            String data = cursor.getString(cursor.getColumnIndex("contact"));
                            itemIds.add(data);
                        } while (cursor.moveToNext());
                    }
                    cursor.close();
                    for (String s : itemIds) {
                        phoneNum = s;
                        if (!phoneNum.equals(prevNumber) && phoneNum != null && ContextCompat.checkSelfPermission(getApplicationContext(),
                                android.Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_DENIED &&
                                ContextCompat.checkSelfPermission(getApplicationContext(),
                                        android.Manifest.permission.SEND_SMS) != PackageManager.PERMISSION_DENIED) {
                            //위치 정보를 전송하기 위해 구글 url에 덧붙인다. 참고로 SMS이기 때문에 주소가 너무 길어지면 안 보내지는 상황이 벌어진다...              
                            textMsg = getString(R.string.accident) + "http://maps.google.com/?q=" + String.valueOf(latitude) + "," + String.valueOf(longitude);
                            try {
                                SmsManager sms = SmsManager.getDefault();
                                sms.sendTextMessage(phoneNum, null, textMsg, null, null);
                            } catch (Exception e) {
                                e.printStackTrace();
                            }
                            Log.d("Message", textMsg + "<" + phoneNum + ">");
                            //이전 번호와 현재 번호를 바꿔준다.
                            prevNumber = phoneNum;
                        }
                    }
                    Toast.makeText(getApplicationContext(), getString(R.string.send_message), Toast.LENGTH_LONG).show();
                    handler.removeCallbacksAndMessages(null);
                    finish();
                }catch (Exception e){
                    Toast.makeText(getApplicationContext(), getString(R.string.error_default), Toast.LENGTH_SHORT).show();
                }
            }
            //약 7초간 응답이 없을 경우 실행된다.
        }, 7000);
    }
}
```
![넘어짐 감지2](/assets/uploads/ajm/smsManager.jpeg)

정상적으로 동작이 이루어지면 위와 같이 문자로 위치 정보가 담겨서 전송되고, url을 클릭하면 주소 정보가 나타나게 된다. 얄루!
<br>

이렇게 안전모 넘어짐 감지 부분에 대해서 정리했다. GPS 측량 학부 수업 때 접했던 피치, 롤, 요를 오랜만에 다시 공부하면서 글을 작성했는데, 역시 그때나 지금이나 어렵다.. 역시 나의 기억의 왜곡이란.. 안전모 넘어짐 감지는 오픈 소스의 적절한? 활용이 아주 잘 이루어진 기능이라고 할 수 있다. 오픈 소스의 위대함을 다시 한 번 상기하며 ~~갑자기?~~ 다음은 안전모 날씨 정보 확인 기능에 대해 정리하는 걸로~

* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :)
- [넘어짐 감지 참고 Github](https://github.com/swift2891/Fall_Detection).
- [오일러 각 참고](https://ko.wikipedia.org/wiki/%EC%98%A4%EC%9D%BC%EB%9F%AC_%EA%B0%81).
- [쿼터니언 회전 참고](http://blog.daum.net/pg365/172).
- [위키피디아 참고](https://en.wikipedia.org/wiki/Euler_angles).
