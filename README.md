# Traffic-Companion
#include <SPI.h>
#include <AIR430BoostETSI.h>
#include <LiquidCrystal.h>
#include <math.h>

double y_g_value;

   
int getproximity();
double getacc_x();
//ultrasonic sensor
const int trigPin = 25;
const int echoPin = 26;
long duration;
int distance;
//accelerometer

const int x_out = A3;
const int z_out = A13 ;
const int y_out = A12 ;
//int x_adc_value;
//double x_g_value;
//const int yInput = A15;
//const int zInput = A4;
//buzzer
const int buzzer =40 ;
//button
const int button =PUSH2 ;

#define CMD_OFF         0
#define CMD_ON          1

//lcd 
// Creates an LCD object. Parameters: (rs, enable, d4, d5, d6, d7)
LiquidCrystal lcd(27,3,4,8,28,34);

//transmitter reciver
struct sControl
{
  unsigned char cmd;
};

struct sControl txControl = { CMD_OFF };      // TX control packet
struct sControl rxControl = { CMD_OFF };      // RX control packet

const int speedlimit = 40;
int Speed;


void setup() 
{
  Serial.begin(9600);
  //Serial.print("hi");
  pinMode(trigPin, OUTPUT); // Sets the trigPin as an Output
  pinMode(echoPin, INPUT); // Sets the echoPin as an Input
  pinMode(buzzer, OUTPUT);
  tone(buzzer,0);

  Radio.begin(0x01, CHANNEL_1, POWER_MAX);

  pinMode(button, INPUT_PULLUP);

  lcd.begin(16, 2);
  lcd.clear();

  //Serial.begin(9600);
  Speed = 90;
  lcd.setCursor(0,0);
  lcd.print("Speed = ");
  lcd.print(Speed);
  lcd.print("km/hr");
  
}


void loop() 
{
  //Serial.println("hi");
  double acc ;
  int proximity= getproximity();
  getacc_x();
  acc=y_g_value;
  Serial.print("ACCELERATION  :  ");
  Serial.println(y_g_value);



if ( Speed <= speedlimit )
{
  if ( proximity < 6 && (y_g_value< -0.10 || y_g_value > 0.30 ) )
  {
    tone (buzzer, 0);
    lcd.setCursor(1,1);
    lcd.print(" RASH DRIVING ");
    tone (buzzer, 200 );
    delay(100) ;
    tone (buzzer, 0);
  }

  else
  {
    for (int i=0;i<16;i++)
    {
    tone (buzzer, 0);
    lcd.setCursor(i,1);
    lcd.print(" ");
   }
  }
}

else
{
  if ( (proximity < 6) || (y_g_value< -0.10 || y_g_value > 0.30 ) ||  ( proximity < 6 && (y_g_value< -0.10 || y_g_value > 0.30 ) ) )
  {
    
    lcd.setCursor(1,2);
    lcd.print(" RASH DRIVING ");
    tone (buzzer, 200 );
    delay(100) ;
    tone (buzzer, 0);
  }

  else
  {
    tone (buzzer, 0);
    for (int i=0;i<16;i++)
    {
    lcd.setCursor(i,1);
    lcd.print(" ");
   }
   
    
  }
}


//lcd.clear();

if (digitalRead(PUSH2) == LOW)
  {
    txControl.cmd = CMD_ON;
    
  }

 else
 {
   txControl.cmd = CMD_OFF;
   //Radio.transmit(ADDRESS_BROADCAST, (unsigned char*)&txControl, sizeof(txControl));
 }
 Radio.transmit(ADDRESS_BROADCAST, (unsigned char*)&txControl, sizeof(txControl));

 /*while (Radio.busy())
  {
    Serial.println("radio busy");
  }
  
  // Turn on the receiver and listen for incoming data. Timeout after 1 seconds.
  // The receiverOn() method returns the number of bytes copied to rxData.
  if (Radio.receiverOn((unsigned char*)&rxControl, sizeof(rxControl), 1000) > 0)
  {
    // Perform action based on incoming command: turn on/off red LED.
    if (rxControl.cmd == CMD_ON)
    {
      tone(3,500);
    }
    else
    {
      tone(3,0);
    }
    
   }

   delay(500);*/

}

  

  
  



int getproximity()
{
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
// Sets the trigPin on HIGH state for 10 micro seconds
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
// Reads the echoPin, returns the sound wave travel time in microseconds
  duration = pulseIn(echoPin, HIGH);
// Calculating the distance
  distance= duration*0.034/2;
  Serial.println("distance ");
  Serial.print(distance);
  Serial.println("");
  return distance;
}

double getacc_x()
{
  
  
  int x_adc_value;
  double x_g_value;
  int z_adc_value;
  double z_g_value;
  int y_adc_value;
  //double y_g_value;
  
  x_adc_value = analogRead(x_out); 
  Serial.print("x = ");
  Serial.print(x_adc_value);
  Serial.print("\t\t");
  x_g_value = ( ( ( (double)(x_adc_value * 3.3)/4096) - 1.65 ) / 0.330 );
 // x_g_value =  ( (double)((x_adc_value-2131) * 0.25));
  Serial.print(x_g_value);
  Serial.print("\t\t");
  z_adc_value = analogRead(z_out); 
  Serial.print("z = ");
  Serial.print(z_adc_value);
  Serial.print("\t\t");
  z_g_value = ( ( ( (double)(z_adc_value * 3.3)/4096) - 1.65 ) / 0.330 );
 // x_g_value =  ( (double)((x_adc_value-2131) * 0.25));
  Serial.print(z_g_value);
  Serial.print("\t\t");
  y_adc_value = analogRead(y_out); 
  Serial.print("y = ");
  Serial.print(y_adc_value);
  Serial.print("\t\t");
  y_g_value = ( ( ( (double)(y_adc_value * 3.3)/4096) - 1.65 ) / 0.330 );
 // x_g_value =  ( (double)((x_adc_value-2131) * 0.25));
  Serial.print(y_g_value);
  Serial.println("\t\t");
  //delay(500);
  //return(y_g_value); 
}
