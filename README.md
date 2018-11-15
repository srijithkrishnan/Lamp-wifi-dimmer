# Lamp-wifi-dimmer
Lamp wifi dimmer
````c
#include <ESP8266WiFi.h>
#include <WiFiUdp.h>

const int ledPin = 9;      // the pin that the LED is attached to


char ssid[] = "........";  //  your network SSID (name)
char pass[] = "1234567890";       // your network password


unsigned int localPort = 2390;      // local port to listen for UDP packets

/* Don't hardwire the IP address or we won't get the benefits of the pool.
 *  Lookup the IP address for the host name instead */
//IPAddress timeServer(129, 6, 15, 28); // in.pool.ntp.org NTP server
IPAddress timeServerIP; // in.pool.ntp.org NTP server address
const char* ntpServerName = "in.pool.ntp.org";

const int NTP_PACKET_SIZE = 48; // NTP time stamp is in the first 48 bytes of the message

byte packetBuffer[ NTP_PACKET_SIZE]; //buffer to hold incoming and outgoing packets

// A UDP instance to let us send and receive packets over UDP
WiFiUDP udp;

void setup()
{
 Serial.begin(115200);
  WiFi.begin("ssid", "password");

  Serial.print("Connecting");
  while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
  }
  Serial.println();

  Serial.print("Connected, IP address: ");
  Serial.println(WiFi.localIP());

}

void loop()
{
/* byte brightness;

  // check if data has been sent from the computer:
  if (Serial.available()) {
    // read the most recent byte (which will be from 0 to 255):
    brightness = Serial.read();
    // set the brightness of the LED:
    analogWrite(ledPin, brightness);
    }*/
  
  //get a random server from the pool
  WiFi.hostByName(ntpServerName, timeServerIP); 

  sendNTPpacket(timeServerIP); // send an NTP packet to a time server
  // wait to see if a reply is available
  delay(1000);
  
  int cb = udp.parsePacket();
  if (!cb) {
    Serial.println("no packet yet");
  }
  else {
    //Serial.print("packet received, length=");
   // Serial.println(cb);
    // We've received a packet, read the data from it
    udp.read(packetBuffer, NTP_PACKET_SIZE); // read the packet into the buffer

    //the timestamp starts at byte 40 of the received packet and is four bytes,
    // or two words, long. First, esxtract the two words:

    unsigned long highWord = word(packetBuffer[40], packetBuffer[41]);
    unsigned long lowWord = word(packetBuffer[42], packetBuffer[43]);
    unsigned long secsSince1900 = highWord << 16 | lowWord;
    const unsigned long seventyYears = 2208988800UL;
    unsigned long epoch = secsSince1900 - seventyYears;
   //Serial.print("The UTC time is ");  // UTC is the time at Greenwich Meridian (GMT)
    int hr1 = ((epoch  % 86400L) / 3600);
    //Serial.print(hr); // print the hour (86400 equals secs per day)
    int min =  ((epoch % 3600) / 60));
    float hr = ((((hr1*60) + min) + 330) / 60);
  
    if(hr >= 12.00 and hr < 18.00)
      analogWrite(ledPin,0);
    else if ( hr >= 18.00 and hr < 23.00) 
      analogWrite(ledPin,255);
    else if ( hr <= 23.00 and hr > 6.00)
      analogWrite(ledPin,0);
    else if ( hr <= 6.00 and hr > 10.00)
      analogWrite(ledPin,64);
    else 
      analogWrite(ledPin,0);
}
 delay(10000);
}
// send an NTP request to the time server at the given address
unsigned long sendNTPpacket(IPAddress& address)
{
  Serial.println("sending NTP packet...");
  // set all bytes in the buffer to 0
  memset(packetBuffer, 0, NTP_PACKET_SIZE);
  // Initialize values needed to form NTP request
  // (see URL above for details on the packets)
  packetBuffer[0] = 0b11100011;   // LI, Version, Mode
  packetBuffer[1] = 0;     // Stratum, or type of clock
  packetBuffer[2] = 6;     // Polling Interval
  packetBuffer[3] = 0xEC;  // Peer Clock Precision
  // 8 bytes of zero for Root Delay & Root Dispersion
  packetBuffer[12]  = 49;
  packetBuffer[13]  = 0x4E;
  packetBuffer[14]  = 49;
  packetBuffer[15]  = 52;

  // all NTP fields have been given values, now
  // you can send a packet requesting a timestamp:
  udp.beginPacket(address, 123); //NTP requests are to port 123
  udp.write(packetBuffer, NTP_PACKET_SIZE);
  udp.endPacket();
  
}
````
