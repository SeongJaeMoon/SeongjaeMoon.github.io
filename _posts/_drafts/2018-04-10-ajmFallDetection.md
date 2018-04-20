---
layout: post
title:  "안전모 프로젝트 리뷰(넘어짐감지부분)"
date:   2018-04-15 23:00:00
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
위와 같이 연락처를 저장하고, 저장된 연락처로 SMS로 위치 정보를 전송할 준비를 완료하였다면, 이제 실제로 넘어짐 감지를 위한 내장 센서를 활용해야 한다. 우선, 안드로이드 스마트폰엔(ios 기종도 마찬가지로) 여러 가지 내장 센서가 탑재되어 있다. 대표적으로 GPS 센서, 압력 센서, 온도 센서, 모션 센서, 기타 등등... 굉장히 많은 센서가 탑재되어 있다. 그중에 모션 센서 부분이 안전모에서 넘어짐 감지를 위해 사용되는 부분이라고 할 수 있다.

먼저, 안드로이드 모션 센서의 종류로는 자이로스코프 센서, 가속도계 센서, 지자기 센서가 있는데, 세 가지 센서의 측정값을 적절하게 조합해서 사용해야 한다. 자세한 내용은 아래서 하나하나 살펴보자.

# Roll, Pitch, Yaw.
우선 스마트폰의 회전 변화를 감지하기 위해서 필수적으로 알아야 하는 롤링, 피칭, 요잉이라는 것에 대해 알이보자.

## Rolling
롤링은 간단하게 말해서 x축에 대한 회전 변화로,

## Pitching


## Yawing


- Roll : X축에 대한 회전 변화. -> 갸우뚱갸우뚱
- Pitch : Y축에 대한 회전 변화. -> 끄덕끄덕
- Yaw : Z축에 대한 회전 변화. -> 도리도리
사실, 위에 내용이 전부다. 이 내용을 수식으로 표현하고, 이 수식을 다시 코드로 작성하는게 ~~헬...~~ 이라고 할 수 있다. 말은 쉬워 보이지만, 꽤나 어려운? 수학적인 요소가 가미되어야 진정으로 스마트폰의 회전 변화를 감지할 수 있다.
## 수식으로 표현.

square root of $$x^2 + y^2 + z^2 = 9.8 $$

$$ \alpha (또는 \psi) $$ - z축.

$$ \gamma (또는 \phi) $$ - x축.

$$ \beta (또는 \theta) $$ - y축.

## 코드로 표현.


# 넘어짐 감지.

# 응답 요구.

# SMS 위치 전송.

* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :)
- [넘어짐 감지 참고 Github](https://github.com/swift2891/Fall_Detection).
- [오일러 각 참고](https://ko.wikipedia.org/wiki/%EC%98%A4%EC%9D%BC%EB%9F%AC_%EA%B0%81).
- [쿼터니언 회전 참고](http://blog.daum.net/pg365/172).
