# Arduino Alarm Clock With Accelerometer
###### By Armaan Mecca and Paul Chapman


##### Project Description
_Guide on how to construct a DIY Arduino alarm clock that "snoozes" with the help of an accelerometer.**ADD MORE**_



### Materials Required
- [Arduino Uno](https://store.arduino.cc/usa/arduino-uno-rev3)
- [16x2 LCD Display](https://www.adafruit.com/product/181)
- [ADXL345 Accelerometer](https://www.adafruit.com/product/1231?gclid=EAIaIQobChMIsuj9gZ2t4gIV1EsNCh2mCAQCEAQYAiABEgLYEvD_BwE)
- [RTC DS1307](https://www.adafruit.com/product/3296?gclid=EAIaIQobChMIy5i0v5it4gIVUIezCh3LIgFfEAQYASABEgJROfD_BwE)
- [100kOhm Variable Resistor](https://www.digikey.com/product-detail/en/bourns-inc/3361P-1-104GLF/3361P-104GLFTR-ND/1088369)
- [220Ohm Resistor](https://www.alliedelec.com/product/rcd-components/cf25-220-jtw/70183313/?gclid=EAIaIQobChMIn5HqwJut4gIVRuDICh2aDgz_EAkYASABEgJfVfD_BwE&gclsrc=aw.ds)
- [3 Push Buttons](https://www.digikey.com.mx/product-detail/en/c-k/PTS645SL43SMTR92-LFS/PTS645SL43SMTR92-LFS-ND/3861373)
- [Buzzer](https://www.radioshack.com/collections/buzzers/products/radioshack-87db-piezo-pulse-buzzer?variant=20332225093)\
**Total Cost:**  $63.27‬ (including Arduino)

### Circuit Diagram

![Circuit Diagram](Alarm CLock.png)

### Simplified Procedure

1. Wire and Solder necessary components onto a breadboard and into the Arduino as shown above
   - LCD
     - RS --> 12
     - en --> 11
     - d4 --> 5
     - d5 --> 4
     - d6 --> 3
     - d7 --> 2
   - RTC
     - SDA --> A4
     - SCL --> A5
     
2. Upload Code into the Arduino(Code Below)
3. Set alarm time
   - Button to Set Alarm is also used to confirm alarm time
4. Wake up on time


### Arduino Code

```
//include libraries
#include <Wire.h>
#include <LiquidCrystal.h>
#include <EEPROM.h>
#include <RTClib.h>
#include <Adafruit_Sensor.h>
#include <Adafruit_ADXL345_U.h>]

Adafruit_ADXL345_Unified accel = Adafruit_ADXL345_Unified(12345);


#define buz 10

//Wire 16x2 LCD display
const int rs = 12, en = 11, d4 = 5, d5 = 4, d6 = 3, d7 = 2;
LiquidCrystal lcd(rs, en, d4, d5, d6, d7);

//Define Global Variables
int tmp,Inc,hor,mIn,add=11; 
int yes = 7;  
int up = 8;   
int mod = 6; 
int off = 0;
int Hor, Min, Sec; 

RTC_DS1307 RTC;

void setup() {
 // Initializes devices and components
 // Prints introductory display
 // Define Accelerometer Serial
 
  #ifndef ESP8266
    while (!Serial);
  #endif
  
  Serial.begin(9600);
  Serial.println("Accelerometer Test"); Serial.println("");
  
  if(!accel.begin()){
    Serial.println("no ADXL345 detected");
    while(1);
  }
  accel.setRange(ADXL345_RANGE_16_G);
  Serial.println("");

  Wire.begin();
  RTC.begin();
  lcd.begin(16,2);
  pinMode(up, INPUT);
  pinMode(yes, INPUT);
  pinMode(mod, INPUT);
  pinMode(buz, OUTPUT);
  digitalWrite(yes, HIGH);
  digitalWrite(mod, HIGH);
  digitalWrite(up, HIGH);
  lcd.setCursor(0,0);
  lcd.print("The Paarm Alarm");
  lcd.setCursor(0,1);
  lcd.print("  Alarm Clock  ");
  delay(2000);
}

void time(){
  // this block enables the user to set the time 
  // based on the RTC time
  // buttons 'up' and 'yes' trigger this block

  // EEProm stores set alarm time --- and not the arduino??? CHECK!

  int tmp=1, mins=0, hors=0, secs=0;
 
  while(tmp==1){
    off = 0;
    if(digitalRead(up)==1){
      Hor++;
      if(Hor==24){
        Hor=0;  
      }
    }
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Set Alarm Time ");
    lcd.setCursor(0,1);
    if(Hor<=9){
      lcd.print("0");
    }
    lcd.print(Hor);
    lcd.print(":");
    lcd.print(Min);
    lcd.print(":");
    lcd.print(Sec);
    delay(200);
    lcd.setCursor(0,2);
    lcd.print("  ");
    lcd.print(":");
    lcd.print(Min);
    lcd.print(":");
    lcd.print(Sec);
    delay(200);
    if(digitalRead(yes)==1){
      hor=Hor;
      EEPROM.write(add++,hor);
      tmp=2;
      while(digitalRead(yes)==0);
    }
  }
  while(tmp==2){
    if(digitalRead(up)==0){
      Min++;
      if(Min==60){ 
        Min=0;
      }
    }
    lcd.setCursor(0,1);
    lcd.print(Hor);
    lcd.print(":");
    if(Min<=9){
      lcd.print("0");
    }
    lcd.print(Min);
    lcd.print(":");
    lcd.print(Sec);
    lcd.print("  ");
    delay(200);
    lcd.setCursor(0,1);
    lcd.print(Hor);
    lcd.print(":");
    lcd.print("  ");
    lcd.print(":");
    lcd.print(Sec);
    lcd.print("  ");
    delay(200);
    
    if(digitalRead(yes)==0){
      mIn=Min;
      EEPROM.write(add++, mIn);
      tmp=0;
      while(digitalRead(yes)==0);
    }
  }
  off = 1;
  delay(1);
}

void Buz(){
  // sets off alarm in 500 millisecond intervals until turned off when RTC time equals stored set time

  
  if(digitalRead(mod)==0){
    off=0;
  }
  
  if(off == 1){
    digitalWrite(buz,HIGH);
    delay(500);

    digitalWrite(buz, LOW);
    delay(500);
  }

  if (off==0) {
    Serial.print("test");
  }

  
}
void TimeCheck(){
  // Compares RTC time to set alarm time and prints .......alarm when times match
  // Also communicates with voiz buzz to set off alarm

  
  int tem[17];
  for(int i=11;i<17;i++){
    tem[i]=EEPROM.read(i);
  }
  if(Hor == tem[11] && Min == tem[12] && off==1){
    add=11;
    Buz();
    Buz();
    lcd.clear();
    lcd.print("alarm...........");
    lcd.setCursor(0,1);
    lcd.print("...........alarm");
    Buz();
    Buz();
  }
}

void loop(){
  // Prints real time on lcd display continuously
  DateTime now = RTC.now();

  
  if(digitalRead(mod) == 0){ 
    current();
    time();
    delay(1000);
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("  Alarm On");
    delay(2000);
    digitalWrite(mod, HIGH);
  }
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Time:");
  lcd.setCursor(6,0);
  if(Hor<=9){
    lcd.print("0");
    lcd.print(Hor=now.hour(),DEC);
  }
  else {
    lcd.print(Hor=now.hour(),DEC);
  }
  lcd.print(":");
  if(Min<=9){
    lcd.print("0");
    lcd.print(Min=now.minute(),DEC);
  }
  else{
    lcd.print(Min=now.minute(),DEC);
  }
  lcd.print(":");
  if(Sec<=9){
    lcd.print("0");
    lcd.print(Sec=now.second(),DEC);
  }
  else{
    lcd.print(Sec=now.second(),DEC);
  }
  lcd.setCursor(0,1);
  lcd.print("Date: ");
  lcd.print(now.month(),DEC);
  lcd.print("/");
  lcd.print(now.day(),DEC);
  lcd.print("/");
  lcd.print(now.year(),DEC);
  TimeCheck();
  delay(500);

  //Turns off Alarm Clock when accelerometer has a certain reading
  sensors_event_t event; 
  accel.getEvent(&event);
  
  Serial.print("Z: "); Serial.print(event.acceleration.z); Serial.print("  ");Serial.println("m/s^2 ");
  delay(500);

  if (event.acceleration.z > -9){
    digitalWrite(buz, LOW);
    off=0;
    
    Serial.print("detected <<<<<<<!!!!");
    Serial.print("\n");
  }

}

void current(){
   // Displays current RTC time
  lcd.setCursor(0,1);
  lcd.print(Hor);
  lcd.print(":");
  lcd.print(Min);
  lcd.print(":");
  lcd.print(Sec);
}
```

#### Code Flow



### Challenges Faced
1. Coding
   -

