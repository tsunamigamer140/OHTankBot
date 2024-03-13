#include <ESP8266WiFi.h>
#include <ESP8266mDNS.h>
#include <WiFiUdp.h>
#include <ArduinoOTA.h>
#include <WiFiClientSecure.h>
#include <NTPClient.h>
#include <UniversalTelegramBot.h>
#include <TimeAlarms.h>

// Wifi network station credentials
String WIFI_SSID = "XXXXXXXXXXX";
String WIFI_PASSWORD = "XXXXXXXXXXXXXXXXX";
// Telegram BOT Token (Get from Botfather)
#define BOT_TOKEN "XXXXXXXXXXXXXXXXXXXXXXXXXX"
#define CHAT_ID "XXXXXXXXXXXXXXX" 
#define CHAT_ID2 "XXXXXXXXXXXXXXX"
#define SAMPLE_SIZE 10

X509List cert(TELEGRAM_CERTIFICATE_ROOT);
WiFiClientSecure secured_client;
UniversalTelegramBot bot(BOT_TOKEN, secured_client);
unsigned long bot_lasttime; // last time messages' scan has been done 

const unsigned long BOT_MTBS = 1000; // mean time between scan messages

const int ledPin = LED_BUILTIN;
const int trigPin = 14; //ultrasonic trig pin value
const int echoPin = 12; //ultrasonic echo pin value
const int humPin = 13; // analog soil moisture sensor pin value
short int autontp=0;
short int timearr[6] ={12, 0, 0, 2, 2, 2024};
short int highest=20;
short int lowest=60;
short int humValue = 0; //stores analog value of humidity pin, 4095-dryest; 0-wettest
short int flag=0;
short int mes1,mes2,i,j;
short int lvl1;
short int tmes[SAMPLE_SIZE];
char tstamp[SAMPLE_SIZE][10];
char tstamp1[SAMPLE_SIZE][10];

String chat_id;
String text;

WiFiUDP ntpUDP;
NTPClient timeClient(ntpUDP, "pool.ntp.org");

void getReply(){ //function to get replies from user in case of reply-based options such as reboot, get_time, set_time etc
  while(!(bot.getUpdates(bot.last_message_received+1))) { };
  chat_id = String(bot.messages[0].chat_id);
  text = bot.messages[0].text;
}

String digitalClockDisplay()
{
  String str="";
  str+=String(hour());
  str+=printDigits(minute());
  str+=printDigits(second());
  str+="\n";

  return str;
}

String printDigits(int digits)
{
  String str="";
  str+=":";
  if(digits < 10)
    str+="0";
  str+=digits;

  return str;
}

int ultrasonic(){ //returns distance as a float from the ultrasonic value
  long duration;
  int distance=0;

  for(int i=0;i<5;i++){
    digitalWrite(trigPin, LOW);
    delayMicroseconds(2);
    digitalWrite(trigPin, HIGH);
    delayMicroseconds(10);
    digitalWrite(trigPin, LOW);

    duration = pulseIn(echoPin, HIGH);
    distance = distance + (int)((duration*0.0343)/2);
  }

  distance = distance/5;

  
  if(distance>lowest){
    lowest=distance;
  }
  else if(distance<highest){
    highest=distance;
  }

  Serial.print("Distance: ");
  Serial.println(distance);
  return distance;
}

int perConv(int dist){
  return (int)(((lowest-dist)/(float)(lowest-highest))*100);
}

void netchange(String chat_id){
  bot.sendMessage(chat_id, "Enter new network ID:","");
  while(!(bot.getUpdates(bot.last_message_received+1))) { };
  chat_id = String(bot.messages[0].chat_id);
  String text = bot.messages[0].text;
  WIFI_SSID=text;

  bot.sendMessage(chat_id, "Enter new network password: ");
  while(!(bot.getUpdates(bot.last_message_received+1))) { };
  chat_id = String(bot.messages[0].chat_id);
  text = bot.messages[0].text;
  WIFI_PASSWORD=text;

  bot.sendMessage(chat_id, "Are you sure you want to reset with SSID: "+WIFI_SSID+" and password: "+WIFI_PASSWORD+" ?.\nEnter 'yes' to confirm and anything else to cancel", "");
  while(!(bot.getUpdates(bot.last_message_received+1))) { };
  chat_id = String(bot.messages[0].chat_id);
  text = bot.messages[0].text;
  String conf=text;

  if(conf.equals("yes")){
    bot.sendMessage(chat_id, "Beginning Network Change", "");
    setup();
  }
  else bot.sendMessage(chat_id, "Network Change Cancelled", "");
}

String canner(){
  Serial.println("Scan start");

  // WiFi.scanNetworks will return the number of networks found
  int n = WiFi.scanNetworks();
  String ns = String(n);
  String fin;
  Serial.println("Scan done");
  if (n == 0) {
      return ("No Networks Found");
  } else {
    fin=(String(n)+" Networks Found: \n");
    for (int i = 0; i < n; ++i) {
      fin+=String(i+1);
      fin+=" : "+WiFi.SSID(i);
      fin+=" ("+String(WiFi.RSSI(i))+") ";
      fin+=(WiFi.encryptionType(i)==AUTH_OPEN ? "O" : "*");
      fin+="\n";
      delay(10);
    }
    return fin;
  }
}



void handleNewMessages(int numNewMessages)
{
  for (int i = 0; i < numNewMessages; i++)
  {
    chat_id = bot.messages[i].chat_id;
    text = bot.messages[i].text;

    String from_name = bot.messages[i].from_name;
    if (chat_id != CHAT_ID && chat_id != CHAT_ID2){
      bot.sendMessage(chat_id, "Unauthorized User!", "");
    }

    if (text == "/check_level")
    {
      int distance = ultrasonic();
      int p = perConv(distance);
      bot.sendMessage(chat_id, "Distance(cm) of water from sensor is: "+String(distance)+"\nPercentage: "+String(p), "");
    }

    else if (text == "/check_motor")
    {
      String hums = String(analogRead(humPin));
      bot.sendMessage(chat_id, "Water Sensor Value is: "+hums, "");
    }

    else if (text == "/change_network")
    {
      netchange(chat_id);
    }

    else if (text == "/network_details")
    {
      bot.sendMessage(chat_id, "SSID: "+WiFi.SSID()+" Strength: "+WiFi.RSSI(), "");
    }

    else if (text == "/network_scan")
    {
      bot.sendMessage(chat_id, canner(), "");
    }

    else if(text == "/calibrate_time"){
      autontp=0;
      setup();
    }
    
    else if(text == "/get_lowest"){
      bot.sendMessage(chat_id,"Lowest tank value(Distance from top) in cm:"+String(lowest),"");
    }

    else if(text == "/get_highest"){
      bot.sendMessage(chat_id,"Highest tank value(Distance from top) in cm:"+String(highest),"");
    }

    else if(text == "/set_time"){
      autontp=1;
      bot.sendMessage(chat_id, "Enter <HH MM SS DD MM YYYY>: ","");
      getReply();
      String str = text;

      bot.sendMessage(chat_id, "Entered Time is:\n"+str+"\nEnter 'YES' to confirm: ","");
      getReply();

      if(text == "YES"){
        
        int StringCount=0;
        while (str.length() > 0){
          int index = str.indexOf(' ');
          if (index == -1) {
            timearr[StringCount++] = str.toInt();
            break;
          }
          else{
            timearr[StringCount++] = str.substring(0, index).toInt();
            str = str.substring(index+1);
          }
        }

        setup();
      }
      else { bot.sendMessage(chat_id, "Exited... ",""); }
    }

    else if (text == "/get_time"){
      bot.sendMessage(chat_id,digitalClockDisplay(),"");
    }

    else if (text == "/reboot"){
      bot.sendMessage(chat_id, "Enter Authentication Code: ","");
      getReply();
      if(text=="ASHTOJO3643"){
        bot.sendMessage(chat_id, "Rebooting now...");
      }
      else {
        bot.sendMessage(chat_id, "Reboot Failed!");
      }
    }

    else{
      String welcome = "Welcome to the Overhead Tank Monitoring System, " + from_name + ".\n";
      welcome += "/check_level\n";
      welcome += "/check_motor\n";
      welcome += "/get_time\n";
      welcome += "/calibrate_time\n";
      welcome += "/set_time\n";
      welcome += "/change_network\n";
      welcome += "/network_details\n";
      welcome += "/network_scan\n";
      bot.sendMessage(chat_id, welcome, "");
    }
  }
}


void setup()
{
  Serial.begin(115200);
  Serial.println("Booting");

  pinMode(ledPin, OUTPUT); // initialize digital ledPin as an output.
  delay(10);
  digitalWrite(ledPin, HIGH); // initialize pin as off (active LOW)
  pinMode(trigPin, OUTPUT); // Sets the trigPin as an Output
  pinMode(echoPin, INPUT); // Sets the echoPin as an Input

  // attempt to connect to Wifi network:
  WiFi.mode(WIFI_STA);
  configTime(I2ST, 330, "pool.ntp.org");      // get UTC time via NTP
  secured_client.setTrustAnchors(&cert); // Add root certificate for api.telegram.org
  Serial.print("Connecting to Wifi SSID ");
  Serial.print(WIFI_SSID);
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  int i=0;
  while (WiFi.status() != WL_CONNECTED)
  {
    i++;
    Serial.print(".");
    delay(5000);
    if(i==10){
      WIFI_SSID = "XXXXXXXXXXXXXXXXX";  //default WIFI SSID incase of /net_change failure
      WIFI_PASSWORD = "XXXXXXXXXXXXXXX"; //default WIFI Key

      ESP.restart();
    }
  }

  ArduinoOTA.setPassword((const char *)"123");

  ArduinoOTA.onStart([]() {
    Serial.println("Start");
  });
  ArduinoOTA.onEnd([]() {
    Serial.println("\nEnd");
  });
  ArduinoOTA.onProgress([](unsigned int progress, unsigned int total) {
    Serial.printf("Progress: %u%%\r", (progress / (total / 100)));
  });
  ArduinoOTA.onError([](ota_error_t error) {
    Serial.printf("Error[%u]: ", error);
    if (error == OTA_AUTH_ERROR) Serial.println("Auth Failed");
    else if (error == OTA_BEGIN_ERROR) Serial.println("Begin Failed");
    else if (error == OTA_CONNECT_ERROR) Serial.println("Connect Failed");
    else if (error == OTA_RECEIVE_ERROR) Serial.println("Receive Failed");
    else if (error == OTA_END_ERROR) Serial.println("End Failed");
  });

  ArduinoOTA.begin();

  Serial.print("\nWiFi connected. IP address: ");
  Serial.println(WiFi.localIP());

  timeClient.begin();
  timeClient.setTimeOffset(19800);

  if(autontp==0){
    timeClient.update();

    time_t epochTime = timeClient.getEpochTime(); //epoch (UNIX) time i.e. time since Jan 1st 1970 considering Time Offset (19800 for IST)

    timearr[0] = timeClient.getHours(); //hours
    timearr[1] = timeClient.getMinutes(); //minutes
    timearr[2] = timeClient.getSeconds(); //seconds
    
    struct tm *ptm = gmtime((time_t*)&epochTime); //time structure to get Day, Month and Year

    timearr[3] = ptm->tm_mday; //day
    timearr[4] = ptm->tm_mon+1; //month 
    timearr[5] = ptm->tm_year+1900; //year
  }

  setTime(timearr[0],timearr[1],timearr[2],timearr[3],timearr[4],timearr[5]);

  bot.sendMessage(CHAT_ID, "Bot On","");
}

void loop()
{
  ArduinoOTA.handle();
  if (millis() - bot_lasttime > BOT_MTBS)
  {
    int numNewMessages = bot.getUpdates(bot.last_message_received + 1);

    humValue = analogRead(humPin);
    

    if(humValue==0){
      if(flag==0){
        String mess = "Motor has been turned on!\nDistance of water is: ";
        flag=1;

        for(i=0;i<SAMPLE_SIZE;i++){
          tmes[i]=0;
          tstamp[i][0]='\0';
          tstamp1[i][0]='\0';
        }
        
        i=0;
        j=0;

        mes1=ultrasonic();
        lvl1=mes1;
        int p = perConv(mes1);
        bot.sendMessage(CHAT_ID,mess+String(mes1)+"\nPercentage: "+String(p),"");
        bot.sendMessage(CHAT_ID2,mess+String(mes1)+"\nPercentage: "+String(p),"");
        delay(1000);
      }

      mes2=ultrasonic();
      if(mes2-mes1==5 && j<SAMPLE_SIZE){

        strcpy(tstamp1[j++],digitalClockDisplay().c_str());
      }

      if(mes2<mes1 && tmes[i]!=1 && i<20){
        i++;
        tmes[i]=1;
        strcpy(tstamp[i],digitalClockDisplay().c_str());
      }
      else if(mes2>mes1 && tmes[i]!=-1 && i<20){
        i++;
        tmes[i]=-1;
        strcpy(tstamp[i],digitalClockDisplay().c_str());
      }
      else if(mes2==mes1 && tmes[i]!=0 && i<20){
        i++;
        tmes[i]=0; 
        strcpy(tstamp[i],digitalClockDisplay().c_str());
      }
      mes1=ultrasonic();
      delay(20); 
    }

    if(humValue==1023 && flag==1){
      int d = ultrasonic();
      int p = perConv(d);
      bot.sendMessage(CHAT_ID,"Motor has been turned off! Level is: "+String(d)+"\nPercentage: "+String(p),"");
      bot.sendMessage(CHAT_ID2,"Motor has been turned off! Level is: "+String(d)+"\nPercentage: "+String(p),"");
      String mess = "Level Change: \n";
      flag=0;

      for(int count=0;count<=i;count++){
        if(tmes[count]==1) { mess+="Increasing\n"; }
        else if(tmes[count]==-1) { mess+="Decreasing\n"; }
        else if(tmes[count]==0) { mess+="Constant\n"; }
      }

      i=0;

      bot.sendMessage(CHAT_ID,mess,"");
      bot.sendMessage(CHAT_ID2,mess,"");

      
      mess = "";
      for(int count=0;count<=j;count++){
        mess += "Level: "+String(lvl1)+"==> Time: "+tstamp1[count]+"\n";
        lvl1 = lvl1+5;
      }
      j=0;

      bot.sendMessage(CHAT_ID,mess,"");
      bot.sendMessage(CHAT_ID2,mess,"");
    }

    while (numNewMessages)
    {
      Serial.println("got response");
      handleNewMessages(numNewMessages);
      numNewMessages = bot.getUpdates(bot.last_message_received + 1);
    }

    bot_lasttime = millis();
  }
}
