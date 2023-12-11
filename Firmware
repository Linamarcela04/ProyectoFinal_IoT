
/*******************************************************************  
*  Firmware final del proyecto del curso Diseño de Dispositivos IoT *
*                                                                   *
*  Por:                                                             *  
*  - Lina Marcela García Palacio                                    *
*  - Juan Pablo Osuna Bonilla                                       *
********************************************************************/


/*Se incluyen las librería necesarias para el programa*/
#include <WiFi.h>
#include <PubSubClient.h>
#include <PZEM004Tv30.h>
#include <math.h>


/* Credenciales para la comunicación a la red WiFi*/
const char* ssid = "ProyectosElectronica";
const char* password = "proyectos";
const char* mqtt_server = "broker.emqx.io";

 
/*Definición de constantes, variables y objetos que se van a utilizar en el programa*/
WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE    (110)
char msg[MSG_BUFFER_SIZE];
int value = 0;

PZEM004Tv30 pzem(Serial2, 16, 17);

/**************************************************************
*            FUNCIÓN PARA CONFIGURAR EL WIFI                  *      
***************************************************************/
void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password); //Se inicializa con la credenciales

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

/**************************************************************
*                   FUNCIÓN DE CALLBACK                       *      
***************************************************************/

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  S+erial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  /*Condicional para enviar un HIGH o LOW al pin de control del módulo relé
   *para accionar el bombillo, con respecto al valor del tópico que ingresa*/
  if ((char)payload[0] == '1') {
    digitalWrite(18, HIGH);  
  } else {
    digitalWrite(18, LOW);  // Turn the LED off by making the voltage HIGH
  }

}

/**************************************************************
*                   FUNCIÓN DE RECONEXIÓN                     *      
***************************************************************/

void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    if (client.connect(clientId.c_str())) {
      Serial.println("connected");
      //Suscripción al tópico "inTopic"
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      delay(5000);
    }
  }
}

/**************************************************************
*                       FUNCIÓN SETUP                         *      
***************************************************************/

void setup()
  //Se configura el pin de control del módulo relé (18) como salida.
  pinMode(18, OUTPUT);
  Serial.begin(115200);
  //Se configura el WiFi
  setup_wifi();
  //Se configura el servidor para mqtt en el puerto 1883
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}

/**************************************************************
*                       FUNCIÓN LOOP                          *      
***************************************************************/

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
  //Se calculan los parámetros de la energía mediante el sensor PZEM-004T
  float voltage = pzem.voltage();
  float current = pzem.current();
  float power = pzem.power();
  float energy = pzem.energy();
  float frequency = pzem.frequency();
  float pf = pzem.pf();
  float pot_apa = voltage*current;
  float pot_act = pot_apa*pf;
  float pot_rea = sqrtf(pow(pot_apa, 2) - pow(pot_act, 2));
 

  if (!isnan(voltage) && !isnan(current) && !isnan(power) && !isnan(energy) && !isnan(frequency) && !isnan(pf) && !isnan(pot_act) && !isnan(pot_rea) && !isnan(pot_apa)) {
   
    snprintf (msg, MSG_BUFFER_SIZE, "V: %.2fV, C: %.2fA, P: %.2fW, E: %.3fkWh, F: %.1fHz, PF: %.2f, PAC: %.2fW, PRAC: %.2fVAR, PAP: %.2fVA ", voltage, current, power, energy, frequency, pf, pot_act, pot_rea, pot_apa);
    snprintf (msg, MSG_BUFFER_SIZE, "P: %.2fW", power);
    Serial.print("Ms: ");
    Serial.println(msg);
    //Conversión de los parámetros calculados a String
    String volt = String(voltage);
    String curr = String(current);
    String powr = String(power);
    String ener = String(energy);
    String frec = String(frequency);
    String pfac = String(pf);
    String pac = String(pot_act);
    String prac = String(pot_rea);
    String pap = String(pot_apa);
    //Envío de los datos al cliente MQTT mediante sus respectivos tópicos
    client.publish("/voltaje", volt.c_str());
    client.publish("/current", curr.c_str());
    client.publish("/power", powr.c_str());
    client.publish("/energy", ener.c_str());
    client.publish("/frequency", frec.c_str());
    client.publish("/pf", pfac.c_str());
    client.publish("/pot_act", pac.c_str());
    client.publish("/pot_rea", prac.c_str());
    client.publish("/pot_apa", pap.c_str());
  }
  delay(5000);
}
