---
layout: post
title:  "아두이노를 활용한 GPS 로그 저장"
date:   2017-12-30 23:00:00
author: Seongjae Moon
categories: Network
tags:   Arduino 아두이노 sketch IoT 사물인터넷
---

아두이노는 오픈 소스를 지향하는 마이크로 컨트롤러(micro controller)를 내장한 기기 제어용 기판으로, 컴퓨터 메인보드의 단순 버전으로 이 기판에 다양한 센서나 부품 등의 장치를 연결할 수 있다. 컴퓨터와 연결해 소프트웨어를 로드하면 동작을 하게 되므로 제어용 전자 장치부터 로봇과 같은 것을 만들 수 있는 **'오픈소스 하드웨어'**라고 할 수 있다.

자유 소프트웨어 운동에서 출발한 오픈 소스라는 개념을 하드웨어 부문까지 확산시킨 것이라 할 수 있겠다. 많은 분야에 이미 아두이노 스케치용 코드가 제공되고 있으며, 개인적으로 다른 분야에 비하면 **꿀잼 영역**이라 할 수 있다. 아두이노에 GPS module을 연결하여 위성데이터를 수신 받고, 수신 받은 데이터를 통해 현재의 위치정보를 PC에 표시하는 것이 가능하다는 정보를 입수하여 **꿀잼 영역**에 도전 해보기로 했다.

#### 위성 데이터
우선, 위성 데이터를 제대로 수신 받기 위해선 각각 최소 4개 이상의 공통 위성에서 측정된 값을 동시에 저장하여 각 측점 간 벡터 기선을 계산해 주어야 한다. 그렇기 때문에 최대한 많은 위성의 신호를 받을 수 있는 시간대와 위치를 선점하여 수신을 실시해야 한다. 현재, 많은 위성이 일정한 간격으로 지구전체를 돌고 있으므로 지구상 어느 곳에서 든지 항상 4개 이상의 GPS 신호를 잡을 수 있다 :)

#### 활용 계획
##### 아두이노의 활용계획은 다음과 같다.
![활용계획](/assets/uploads/arduinoPlan.jpg)
##### 아두이노 활용계획을 세웠으므로, 아두이노가 필요하다!
![아두이노 키트](/assets/uploads/arduinoKitSet.jpg)
위 사진은 아두이노 우노 R3 키트와 GPS모듈 및 모듈을 연결하기 위한 여러 가지 회선과 PC와 연결할 수 있는 USB젠더가 들어있는 모습이다. 얄루:)  (용산 전자상가에서 구매하거나, 인터넷에서 손 쉽게 구매할 수 있다.)
#### 스펙


아두이노 우노 R3 |GPS Module
-------------------|----------------------------------
마이크로 컨트롤러: ATmega328|동작 전압: 3.3v/5v
클락 주파수: 16MHz|통신 방식: UART(시리얼통신)
EEPROM 1KB|BAUD RATE: 9600
SRAM: 2KB|기타 특징: Ublock, NMEA 프로토콜 지원, 소비전력이 적음, 그로브 인터페이스 지원
플래시 메모리: 32KB|
디지털 입출력 핀: 14개|
아날로그 입출력 핀: 6개|
동작 전압: 5V|
추천 입력 전압: 7V ~ 12V|


위의 자료는 아두이노 우노R3와 GPS모듈의 기본적인 스펙이다. GPS모듈은 시리얼 통신으로 9600bps로 통신을 하는 것을 확인 할 수 있다.
#### 아두이노 스케치 다운로드
[아두이노 홈페이지](www.arduino.cc)에선 아두이노를 제어하기 위한 통합개발환경인 아두이노 스케치를 무료로 다운 받을 수 있도록 제공한다. 사용할 버전에 맞춰 다운로드를 하면 이로써 기본적인 준비는 완료된다.  

#### GPS 모듈 연결
아두이노와 GPS 모듈을 연결하기 전에 GPS 모듈과 아두이노 키트에 들어있던 연결선을 납땜해주자. 납땜을 하지 않고 데이터를 받게 되면, 회선의 연결 상태 불능으로 위성데이터 수신의 어려움이 있을 수 있어 납땜을 해주는 것이 좋다.
![납땜납땜!](/assets/uploads/arduinoGPS.jpg)

납땜이 완료되면, 아두이노와 GPS모듈의 연결 해야한다. 아래 그림처럼 각각의 배선을  연결하면 된다.
##### (좌) GPS 모듈, (우) 아두이노
- VCC -> 5v
- GND -> GND
- RX -> 3번 PIN
- TX -> 2번 PIN
![아두이노 모듈](/assets/uploads/arduinoGPSmodule.jpg)

#### GPS 위성데이터 수신
이제 모든 준비가 끝났으니, 실제 데이터를 수신받도록 한다. 위성데이터는 여러가지 데이터를 포함하지만, 여기선 NMEA0183 라고 하는 프로토콜 규약에 의해서 수신 받은 데이터를 표시하게 된다. 내가 수신 받을 데이터에 대한 기본적인 정보는 알고 넘어가는게 좋을테니, NMEA0183 프로토콜에 대해 간단하게 알아보도록 하자.

NMEA0183란 시간, 위치, 방위 등의 정보를 전송하기 위한 규격이다. NMEA0183은 미국의 **The National Marine Electronics Association**에서 정의해 놓았다. **이 데이터들은 주로 자이로컴퍼스, GPS, 나침반, 관성항법장치(INS)에 사용된다.** NMEA0183은 ASCII와 직렬 방식의 통신을 사용한다.  

**NMEA의 기본적인 규칙으로는 $로 시작한다, 첫 두 자리는 제품의 종류를 나타낸다.** GPS 제품일 경우 GP, 수심 측정 장비인 Depth Sounder 제품일 경우 SD 를 사용한다, 다음 세 자리는 해당 프로토콜이 가지고 있는 데이터의 종류를 나타낸다, 데이터의 구분은 ','로 한다, '*'로 끝난다. '$'와 '*'사이의 모든 데이터를 exclusive or 연산을 하여 체크섬을 만들어 추가한다. 라는 주요 규약 있으며, 다른 세부 규약 등이 있다.

그렇다.. 뭐가 굉장히 복잡하다.. 일단은 저러한 패턴으로 데이터를 수신 받는다는 정도로만 알고 있으면 될성싶다. 아래 그림은 야외 나들이 기분을 살리며, 나의 보물 1호 삼성 랩탑과 ~~삼성 광고 아니다~~ 아두이노를 연결하여 GPS 데이터를 수신 받기 위해 준비 중인 모습이다.
![나들이갈까-볼빨간 사춘기](/assets/uploads/arduinoSetGPS.jpg)

이제 아두이노 소스를 살펴 볼 차례이다. 아두이노는 아두이노 스케치를 통해 코드를 업로드 하고 나면, loop 작업을 전원이 공급되는 동안 무한히 반복한다고 할 수 있다.
```c
#include <SoftwareSerial.h>
SoftwareSerial GPS(2,3);

void setup(){
	GPS.begin(9600);
	Serial.begin(9600);
}
void loop(){
	if(Serial.available()){
		GPS.write(Serial.read());
	}
	if(GPS.available()){
		Serial.write(GPS.read());
	}
}
```
위 코드는 GPS 데이터를 수신 받기 위해 쓰이는 가장 간단한 코드라고 할 수 있다. 9600bps의 속도로 데이터를 수신 받도록 하고, GPS에서 받은 데이터를 아두이노 스케치의 시리얼 모니터(단축키: Ctrl+Shift+M)로 보도록 이루어져있다.
![시리얼 모니터](/assets/uploads/arduinoSerialM1.jpg)
위 사진처럼 시리얼 모니터로 측정 되는 데이터가 보이게 되면 정상이다. 이러한 데이터는 NMEA 프로토콜로 이루어졌기 때문에, 알아보기가 굉장히 힘들다. 열심히 구글링 해 본 결과, 경위도 좌표로 변환된 데이터를 시리얼 모니터로 모니터링 할 수 있다. 얄루! :) 우선 코드를 살펴 보자.
```c
#include <SoftwareSerial.h>
#include <TinyGPS.h>
 
// Arduino에서 사용할 핀을 정의하여 선언
#define RXPIN 6
#define TXPIN 5
//이 값을 GPS의 보오율과 동일하게 설정(9600)
#define GPSBAUD 9600
 
// TinyGPS 객체의 인스턴스를 생성 -> TinyGPS 라이브러리 패키지를 다운로드 받아야 함!
TinyGPS GPS;
// 위에서 정의한 핀으로 NewSoftSerial 라이브러리를 초기화한다.
SoftwareSerial uart_gps (RXPIN, TXPIN);
 
// 여기서 함수의 프로토 타입을 선언
// TinyGPS 라이브러리를 사용
void getgps (TinyGPS & gps);
 
// setup 함수에서 두 개의 직렬 포트를 초기화
// 표준 하드웨어 직렬 포트 (Serial ())를 사용하여 수신.
void setup ()
{
  Serial.begin (9600);
  uart_gps.begin (GPSBAUD);
  
  Serial.println ( "");
  Serial.println ( "... 수신 대기 중 ...");
  Serial.println ( "");
}
 
// 코드의 메인 루프, 단지 데이터가 유효한지 확인하는 것뿐!
// ardiuno의 RX 핀은 데이터가 유효한 NMEA 문장인지 확인
// getgps () 함수로 실제 데이터 출력
void loop ()
{
  while (uart_gps.available ()) // RX 핀에 데이터가있는 동안 ...
  {
      int c = uart_gps.read (); // 데이터를 변수에로드 ...
      if (gps.encode (c)) // 새로운 유효한 문장이있는 경우 ...
      {
        getgps (gps); // 데이터를 가져온다.
      }
  }
}
 
// getgps 함수는 우리가 원하는 값을 얻어서 출력
void getgps (TinyGPS & gps)
{
  // 모든 데이터를 코드에서 사용할 수있는 varialbes로 가져 오려면, 변수를 정의하고 객체를 핸들링 하면된다.
  // 데이터. 함수의 전체 목록을 보려면에서 keywords.txt 파일을 참조
  // TinyGPS와 NewSoftSerial 라이브러리.
  // 경, 위도 변수를 정의
    float latitude, longitude;
  // 함수 호출
  gps.f_get_position (&latitude, &longitude);
  //경위도 출력 가능
  Serial.print("Lat/Long: ");
  Serial.print(latitude,5);
  Serial.print(", ");
  Serial.println(longitude,5);
  
  // 날짜와 시간은 같음
  int year;
  int year;
  byte month, day, hour, minute, second, hundredths;
  gps.crack_datetime(&year,&month,&day,&hour,&minute,&second,&hundredths);
  // 데이터 및 시간 출력
  Serial.print("Date: "); Serial.print(month, DEC); Serial.print("/");
  Serial.print(day, DEC); Serial.print("/"); Serial.print(year);
  Serial.print("  Time: "); Serial.print(hour, DEC); Serial.print(":");
  Serial.print(minute, DEC); Serial.print(":"); Serial.print(second, DEC);
  Serial.print("."); Serial.println(hundredths, DEC);

//고도와 코스 값을 직접 출력
  Serial.print("Altitude (meters): "); Serial.println(gps.f_altitude());  Serial.print("Course (degrees): "); Serial.println(gps.f_course());
  Serial.print("Speed(kmph): "); Serial.println(gps.f_speed_kmph());
  Serial.println ();
  
  //통계 값 출력
 unsigned long chars;
  unsigned short sentences, failed_checksum;
  gps.stats(&chars, &sentences, &failed_checksum);
 delay (10000);
}
```
이렇게 코드를 작성하고 나면,  시리얼 모니터에서 비교적 보기 좋게 나오는 것을 확인 할 수 있다! 이제 이 값을 가지고 구글어스에 뿌려보도록 하자.
![노트패드](/assets/uploads/arduinoSerialM2.jpg)
우선 구글 어스에서 내 gps 로그를 보기위해선, 당연히 [구글어스](https://www.google.co.kr/intl/ko/earth/download/gep/agree.html)가 설치 되어 있어야 한다. 또한, 시리얼 모니터에서 본 GPS로그 데이터를 .gpx형식으로 변환해주는 작업이 필요하다. (gpx 포멧에 대한 더 자세한 설명은 [여기를 참고](https://en.wikipedia.org/wiki/GPS_Exchange_Format)) 이러한  .gpx형식의 데이터는 xml형식으로 되어있기 때문에 xml형식으로 Notepad++의 매크로 기능을 이용해 적절하게 형식을 맞춰 바꿔주면 끝!

구글어스 프로그램을 실행하고, 파일> 열기 > Gps > 내 gpx 파일 선택을 하면 된다.
![구글어스 보기](/assets/uploads/arduinoGoogleEarth.jpg)

아주 잘 나온다! 얄루 :)

간단하게? 아두이노를 이용한 GPS 측정 테스트를 해볼 수 있었다. 개개인은 수신 받는 데이터가 원시 데이터가 아닌 가공 및 처리된 데이터를 이용하게 되므로 그 자체로만 판단하게 되지만, **원시 데이터를 사용자가 원하는 수준으로 편리하게 이용할 수 있도록 변환하는 것에는 굉장히 많은 작업과 시간이 소요됨을 배울 수 있었다.**

꿀잼 영역이라 생각했지만.. 역시 피곤하다.. 다음은 아침마다 일어나지 못해 고생하는 나를 위해 알람시계를 만들어 보는걸로~

* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :)
* [아두이노와 GPS 모듈 연결](http://blog.naver.com/roboholic84/220752975675)
* [아두이노 GPS 로그 표시](http://deneb21.tistory.com/331)
* [NMEA](https://ko.wikipedia.org/wiki/NMEA)<br>
김동문(2012), GPS 측량 및 활용 pp.153~155. pp.200~204.
