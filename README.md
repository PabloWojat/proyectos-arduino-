# proyectos-arduino-
Se depositaran los proyectos arduino que resultan interesantes. 
#include <Servo.h>
#include <ESP8266WiFi.h>
#include <ESP8266WiFiMulti.h>
#include <WiFiUdp.h>

ESP8266WiFiMulti wifiMulti; // Crea una instancia de la clase ESP8266WiFiMulti, llamada 'wifiMulti'

WiFiUDP UDP; // Crea una instancia de la clase WiFiUDP para enviar y recibir

IPAddress timeServerIP; // dirección del servidor NTP time.nist.gov
const char * NTPServerName = "time.nist.gov";

const int NTP_PACKET_SIZE = 48; // La marca de tiempo NTP está en los primeros 48 bytes del mensaje

byte NTPBuffer [NTP_PACKET_SIZE]; // buffer para contener paquetes entrantes y salientes

Servo myservo; // crea un objeto servo para controlar un servo
// se pueden crear doce servo objetos en la mayoría de los tableros

int pos = 0; // variable para almacenar la posición del servo

configuración nula () {
  myservo.attach (5); // adjunta el servo en el pin 5, también conocido como D1, al objeto servo

  // abre la tapa por defecto
    Serial.println ("abrir la tapa");
    for (pos = 95; pos> = 0; pos - = 1) {// va de 95 grados a 0 grados
      myservo.write (pos); // dile al servo que vaya a la posición en la variable 'pos'
      retraso (15); // espera 15 ms para que el servo alcance la posición
    }
    
  Serial.begin (115200); // Inicie la comunicación en serie para enviar mensajes a la computadora
  retraso (10);
  Serial.println ("\ r \ n");

  startWiFi (); // Intenta conectarte a algunos puntos de acceso dados. Entonces espera una conexión

  startUDP ();

  if (! WiFi.hostByName (NTPServerName, timeServerIP)) {// Obtenga la dirección IP del servidor NTP
    Serial.println ("Falló la búsqueda de DNS. Reinicio");
    Serial.flush ();
    ESP.reset ();
  }
  Serial.print ("IP del servidor de hora: \ t");
  Serial.println (timeServerIP);
  
  Serial.println ("\ r \ nEnviando solicitud NTP ...");
  sendNTPpacket (timeServerIP);  
}

intervalo largo sin signo NTP = 60000; // Solicitar hora NTP cada minuto
unsigned long prevNTP = 0;
unsigned long lastNTPResponse = millis ();
uint32_t timeUNIX = 0;

unsigned long prevActualTime = 0;

bucle vacío () {
  unsigned long currentMillis = millis ();

  if (currentMillis - prevNTP> intervalNTP) {// Si ha pasado un minuto desde la última solicitud de NTP
    prevNTP = currentMillis;
    Serial.println ("\ r \ nEnviando solicitud NTP ...");
    sendNTPpacket (timeServerIP); // Enviar una solicitud NTP
  }

  uint32_t time = getTime (); // Compruebe si ha llegado una respuesta NTP y obtenga el tiempo (UNIX)
  if (time) {// Si se ha recibido una nueva marca de tiempo
    timeUNIX = tiempo;
    Serial.print ("Respuesta NTP: \ t");
    Serial.println (timeUNIX);
    lastNTPResponse = currentMillis;
  } else if ((currentMillis - lastNTPResponse)> 3600000) {
    Serial.println ("Más de 1 hora desde la última respuesta NTP. Reinicio");
    Serial.flush ();
    ESP.reset ();
  }

  uint32_t actualTime = timeUNIX + (currentMillis - lastNTPResponse) / 1000;
  uint32_t easternTime = timeUNIX - 18000 + (currentMillis - lastNTPResponse) / 1000;
  if (actualTime! = prevActualTime && timeUNIX! = 0) {// Si ha pasado un segundo desde la última impresión
    prevActualTime = actualTime;
    Serial.printf ("\ rUTC time: \ t% d:% d:% d", getHours (actualTime), getMinutes (actualTime), getSeconds (actualTime));
    Serial.printf ("\ rEST (-5): \ t% d:% d:% d", getHours (easternTime), getMinutes (easternTime), getSeconds (easternTime));
    Serial.println ();
  } 

  // 7:30 am
  if (getHours (easternTime) == 7 && getMinutes (easternTime) == 30 && getSeconds (easternTime) == 0) {
  //abre la tapa
    Serial.println ("abrir la tapa");
    for (pos = 95; pos> = 0; pos - = 1) {// va de 95 grados a 0 grados
      myservo.write (pos); // dile al servo que vaya a la posición en la variable 'pos'
      retraso (15); // espera 15 ms para que el servo alcance la posición
    }    
  }

    

  // medianoche
  if (getHours (easternTime) == 0 && getMinutes (easternTime) == 0 && getSeconds (easternTime) == 0) {
    //cerrar la tapa
    Serial.println ("cerrando la tapa");
    for (pos = 0; pos <= 95; pos + = 1) {// va de 0 grados a 95 grados
     // en pasos de 1 grado
      myservo.write (pos); // dile al servo que vaya a la posición en la variable 'pos'
      retraso (15); // espera 15 ms para que el servo alcance la posición
    }    
  }   

/ *
// pruebas
  if (getHours (easternTime) == 12 && getMinutes (easternTime) == 45 && getSeconds (easternTime) == 0) {
    //cerrar la tapa
    Serial.println ("cerrando la tapa");
    for (pos = 0; pos <= 95; pos + = 1) {// va de 0 grados a 95 grados
     // en pasos de 1 grado
      myservo.write (pos); // dile al servo que vaya a la posición en la variable 'pos'
      retraso (15); // espera 15 ms para que el servo alcance la posición
    }
  //abre la tapa
    Serial.println ("abrir la tapa");
    for (pos = 95; pos> = 0; pos - = 1) {// va de 95 grados a 0 grados
      myservo.write (pos); // dile al servo que vaya a la posición en la variable 'pos'
      retraso (15); // espera 15 ms para que el servo alcance la posición
    }    
  }
 * / 
}

void startWiFi () {// Intenta conectarte a algunos puntos de acceso dados. Entonces espera una conexión
  wifiMulti.addAP ("ssid_from_AP_1", "your_password_for_AP_1"); // agrega redes Wi-Fi a las que deseas conectarte
  //wifiMulti.addAP("ssid_from_AP_2 "," your_password_for_AP_2 ");
  //wifiMulti.addAP("ssid_from_AP_3 "," your_password_for_AP_3 ");

  Serial.println ("Conexión");
  while (wifiMulti.run ()! = WL_CONNECTED) {// Espera a que se conecte el Wi-Fi
    retraso (250);
    Serial.print ('.');
  }
  Serial.println ("\ r \ n");
  Serial.print ("Conectado a");
  Serial.println (WiFi.SSID ()); // Dinos a qué red estamos conectados
  Serial.print ("dirección IP: \ t");
  Serial.print (WiFi.localIP ()); // Enviar la dirección IP del ESP8266 a la computadora
  Serial.println ("\ r \ n");
}

void startUDP () {
  Serial.println ("Iniciando UDP");
  UDP.begin (123); // Comience a escuchar mensajes UDP en el puerto 123
  Serial.print ("Puerto local: \ t");
  Serial.println (UDP.localPort ());
  Serial.println ();
}

uint32_t getTime () {
  if (UDP.parsePacket () == 0) {// Si no hay respuesta (todavía)
    devuelve 0;
  }
  UDP.read (NTPBuffer, NTP_PACKET_SIZE); // lee el paquete en el búfer
  // Combina los 4 bytes de la marca de tiempo en un número de 32 bits
  uint32_t NTPTime = (NTPBuffer [40] << 24) | (NTPBuffer [41] << 16) | (NTPBuffer [42] << 8) | NTPBuffer [43];
  // Convertir la hora NTP en una marca de tiempo UNIX:
  // El tiempo Unix comienza el 1 de enero de 1970. Eso es 2208988800 segundos en tiempo NTP:
  const uint32_t setenta años = 2208988800UL;
  // restar setenta años:
  uint32_t UNIXTime = NTPTime - setenta años;
  return UNIXTime;
}

void sendNTPpacket (dirección IP y dirección) {
  memset (NTPBuffer, 0, NTP_PACKET_SIZE); // establece todos los bytes en el búfer a 0
  // Inicializa los valores necesarios para formar una solicitud NTP
  NTPBuffer [0] = 0b11100011; // LI, Versión, Modo
  // envía un paquete solicitando una marca de tiempo:
  UDP.beginPacket (dirección, 123); // Las solicitudes NTP son al puerto 123
  UDP.write (NTPBuffer, NTP_PACKET_SIZE);
  UDP.endPacket ();
}

inline int getSeconds (uint32_t UNIXTime) {
  devuelve UNIXTime% 60;
}

inline int getMinutes (uint32_t UNIXTime) {
  return UNIXTime / 60% 60;
}

inline int getHours (uint32_t UNIXTime) {
  return UNIXTime / 3600% 24;
}
