---
layout: post
title:  "안드로이드 Activity의 생명주기"
date:   2017-12-04 10:10:00
author: Seongjae Moon
categories: Android
tags:   안드로이드 Activity View
---

안드로이드의 Acitivity는 각 상황에 맞는 독특한 생명주기를 갖는다. 사용자의 사용으로 어플리케이션의 UI가 계속해서 화면전환이 일어남에 따라 그때그때 필요한 메소드가 호출 되며, 오버라이딩해서 사용하게 된다.

현재는 Activity와 약간 다른 생명주기를 갖는 Fragment를 많이 사용하기 위해 공부 중이지만(너무 늦은감이 있다… 다음 안드로이드 정리는 Fragment를 정리하는 걸로~) Task관리와 더불어 화면전환이 일어날 때 리스너 해제나 상태 값 저장 등이 꼭 필요한 경우가 있다. 이럴 경우 각 생명주기에 맞춰 메소드 호출이 꼭 필요하다.

## Acitivity는 크게 7가지 상태 변화를 가지며 Context 클래스를 상속받는다.

![안드로이드 Document 참고](/assets/uploads/ActivityLife.jpg)

### 1.onCreate

onCreate는 Acitivity가 처음 실행 되는 상태에 제일 먼저 호출되는 메소드로 여기에 실행시 필요한 각종 초기화 작업을 적어준다. 기본적으로 내가 메소드를 정의하지 않더라도 안스에서 Acitivity를 생성하면 자동으로 생성된다.
```java
@Override   
public void onCreate(Bundle savedInstanceState){
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    /*
    요 Bundle 타입의 객체를 저장하는 메소드는 사용자의 Back 버튼 클릭이나 finish() 메소드를 통한 액티비티 인스턴스의 소멸이 아닌
    시스템의 제약 조건(ANR과 같은 상황이 아닐까 생각된다.)으로 인해 액티비가 소멸될 경우 인스턴스 자체는 소멸되지만 상태를 저장하고 있어야 하는 친구가 필요하다.
    그 상태 값을 Bundle 클래스의 키-값 쌍으로 가지고 저장해주는 친구가 바로 요 메소드라고 할 수 있다.
    자세한 내용은 밑에 url을 첨부한다.
    */

    //여기다가 Activity 생성시 필요한 코드들을 작성한다.
    /*
      각종 View 초기화 ex. button = (Button)findViewById(R.id.btn);
      사실 갓 ButterKnife를 이용해서 이렇게 사용하지 않고도 편하게 View를 초기화하고 각종 리스너 등록이 가능하다.
     */      
    //일반적으로 객체의 초기화나 변수 초기화 등의 모든 UI를 비롯한 실행하기 전 꼭 필요한 초기화 작업을 실행한다.
}
```

### 2.onStart

onStart는 생명주기상 onCreate 다음에 호출된다. ~~사실 나는 자주 쓰진 않는다.~~ 보통 회원가입 등이 필요한 기능에서 리스너 객체 등을 onCreate에서 선언하고 onStart에서 선언된 리스너를 등록한 후, 이미 로그인 된 사용자인지를 구분하여 로그인 화면으로 넘어가지 않고 바로 메인으로 넘어가게 할 때 사용한다. 브로드캐스트리시버를 사용할 때도 보통 여기다가 등록한다! onStop과 짝을 이루고 다루게 된다.
```java
@Override
public void onStart(){
    super.onStart();
  //각종 리스너 등록
  /*
    ex. registerReceiver(receiver, filter);
  */
}
```

### 3.onResume

onResume는 생명주기상 onStart 다음에 호출된다. 안드로이드 Document엔 사용자와 상호작용이 가능할 때 호출이 된다고 나와있다. onPause와 더불어 onCreate 다음으로 가장 많이 사용되는 메소드이다. 액티비티는 사용자에 요구사항에 따라 계속해서 변화하게 된다. onCreate와 onStart가 사용자와 상호작용 하기 전과 직전에 일어난다면 onResume는 직접적으로 사용자의 터치 이벤트나 Toast등이 동작할 수 있을 때 호출된다고 생각된다.
```java
@Override
public void onResume(){
    super.onResume();
  //사용자에게 보여질 데이터 등 가져오기
        ex.
        /*
        Gson gson = new Gson();
        String json = sharedPreferences.getString("data", "");
        data = gson.fromJson(json, Data.class);
        */
        //위 코드는 밑에 onPause에서 자세하게 설명한다.
}
```

### 4.onPause

onPause는 생명주기상 사용자에게 보여지지 않을 때 호출된다. onResume과 짝을 이루며, 사용자가 Home 버튼을 클릭해서 액티비티가 보여지지 않거나 다른 액티비티가 보여질 경우, 각종 View나 데이터들을 임시로 저장해야할 필요가 있을 때가 있다. 이 때 Application Context를 사용해서 저장하거나 Preference를 통해서 값을 저장하게 되는데, 이러한 코드들을 여기에 작성 하면된다. 또한, 카메라를 사용하거나 위치 정보 등을 사용할 경우 리스너나 자원 등을 해제할 때도 onPause 밑에 작성해주면 되겠다.
```java
 @Override
 public void onPause(){
     super.onPause();
 //사용자에게 보여지지 않을 때 임시로 뭔가 저장하거나 자원 해제 등 작성
     ex.
     /*
     SharedPreferences.Editor prefsEditor = sharedPreferences.edit();
     Gson gson = new Gson();
     String json = gson.toJson(data);
     prefsEditor.putString("data", json);
     prefsEditor.apply();
     */

     /*
     위 코드는 갓 Google의 (구글만세!) gson이라는 API를 통해 json데이터를 핸들링하는 코드이다.
     data라는 뭔지 모를 예제 데이터를 스트링 형식으로 저장한다.
     prefsEditor라는 추상 객체에 스트링 형식으로 json 이라는 스트링 참조값이 저장 된다.
     apply()로 disk에 쓰기.
     */
}
```

### 5.onStop

onStop는 생명주기상 액티비티가 보여지지 않을 때 호출된다. onStart와 짝을 이루며, onPause와 비슷해 보이는데 onPause보다 나중에 호출된다. 가끔은 onPause와 onStop 중에 어디다 코드를 작성 해야할지 고민이 될 때가 있다. ~~그럴 땐 일단 구글링부터~~ 보통 객체의 null 체크 후에 값이 있을 경우 자원을 해제할 때 사용한다.
```java
@Override
public void onStop(){
    super.onStop();
//사용자에게 액티비티가 보여지지 않을 때 호출된다.
    ex.
    /*
       if (tts!= null) {
            tts.stop();
        }
    */

    /*
        위 코드는 tts라는 객체가 null이 아닐 경우 stop()이라는 메소드를 호출하라는 코드이다.
    */
}
```

### 6.onRestart

onRestart는 생명주기상 onStop 됐다가 다시 보여질 때 호출된다. ~~배움이 모자라서 이 친구를 써본적이 없다..~~ 안드로이드 Document를 살펴보니 onStop과 onRestart는 복잡한 작업이 아닐 경우 onPause와 onResume만으로도 해결이 가능한 것으로 보인다 ㅎㅎ 단, “onStop() 전에 onPause() 메서드가 호출되기는 하지만, 데이터베이스에 정보를 쓰는 작업과 같이 규모가 크고 CPU를 많이 사용하는 종료 작업을 수행하는 경우 onStop()을 사용해야 합니다.”라고 안드로이드 Document에 적혀 있는 것을 확인 할 수 있다.
```java
@Override
public void onRestart(){
    super.onRestart();
}
```

### 7.onDestroy

onDestroy는 생명주기상 액티비티가 시스템에서 소멸될 때 호출된다. onStop이나 onPause에서 대부분에 자원을 해제했겠지만(안 해주면 여러가지 문제점이 발생한다.), 마지막으로 자원을 정리하거나 액티비티 소멸시 간단한 노티피케이션 등을 뛰울때 사용할 수 있다.
```java
Override
public void onDestroy(){
    super.onDestroy();
    //액티비티가 소멸될 때 호출할 메소드등 선언된
    ex.
    /*
      if (tts != null) {
        tts.stop();
        tts.shutdown();
    }
    */
    /*
    위 onStop()에선 stop만 했다면, onDestroy에서는 shutdown()메소드 까지 호출하여 완전히 자원을 해제해준다.
    */
}
```
안드로이드 4대 컴포넌트 중 하나인 Activity에 생명주기에 대해 간단하게 알아봤다. 사실, 100 퍼센트 호출된다는 보장이 없다는 글을 본적이 있다.. 아무튼, 코딩을 하면서 이러한 **생명주기를 잘 생각하면서** 코딩하는 습관이 필요하다고 할 수 있겠다.

다음 목표는 java 코드가 아닌, Kotiln!! 코드를 이용한 Fragment에 대해 전반적인 내용을 다루는 걸로~

* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :)
안드로이드 Acitivity에 관한 자세한 내용은 : [안드로이드 Document](https://developer.android.com/reference/android/app/Activity.html#ActivityLifecycle)
* onSaveInstanceState에 관한 자세한 내용은 : [안드로이드 Document](https://developer.android.com/training/basics/activity-lifecycle/recreating.html?hl=ko)
