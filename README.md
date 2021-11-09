# Laboratorio_IOT
Tercer Laboratorio - Franky Achagua, Luz Benavides
#include <Arduino.h>
#include <WiFi.h>
#include <PubSubClient.h>



//**************************************
//*********** MQTT CONFIG **************
//**************************************
const char *mqtt_server = "node02.myqtthub.com";
const int mqtt_port = 1883;
const char *mqtt_user = "esp32";
const char *mqtt_pass = "esp32";
const char *root_topic_subscribe = "Temperatura/esp32";
const char *root_topic_publish = "Temperatura/public_esp32";


//**************************************
//*********** WIFICONFIG ***************
//**************************************
const char* ssid = "LEITON";
const char* password =  "103/A_202003921_12/A90.12";



//**************************************
//*********** GLOBALES   ***************
//**************************************
WiFiClient espClient;
PubSubClient client(espClient);
char msg[25];
long count=0;


//************************
//** F U N C I O N E S ***
//************************
void callback(char* topic, byte* payload, unsigned int length);
void reconnect();
void setup_wifi();

void setup() {
  
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);
  pinMode(21, OUTPUT);  //Pin8 (LED_Humedad) Como salida
  pinMode(22, OUTPUT);  //Pin8 (Ventilador) Como salida
  pinMode(36, INPUT);  //Pin36 (Sensor_Humedad_Y_Temperatura) Como entrada
  
}

void loop() {

  digitalWrite(22, HIGH);
  delay(5000);
  digitalWrite(21, HIGH);
  delay(5000);
  int Temperatura = analogRead(36);
  Serial.print("La Temperatura es: ");
  Serial.println( Temperatura);
  delay(5000);
  
  if (!client.connected()) {
    reconnect();
  }

  if (client.connected()){
    String str = "La cuenta es -> " + String(count);
    str.toCharArray(msg,25);
    client.publish(root_topic_publish,msg);
    count++;
    Serial.println(msg);
    delay(5000);
  }
  client.loop();
}




//*****************************
//***    CONEXION WIFI      ***
//*****************************
void setup_wifi(){
  delay(5000);
  // Nos conectamos a nuestra red Wifi
  Serial.println();
  Serial.print("Conectando a ssid: ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("Conectado a red WiFi!");
  Serial.println("Dirección IP: ");
  Serial.println(WiFi.localIP());
}



//*****************************
//***    CONEXION MQTT      ***
//*****************************

void reconnect() {

  while (!client.connected()) {
    Serial.print("Intentando conexión Broker...");
    // Creamos un cliente ID
    String clientId = "Micro_Esp";
    
    // Intentamos conectar
    if (client.connect(clientId.c_str(),mqtt_user,mqtt_pass)) {
      Serial.println("Conectado al broker!");
      // Nos suscribimos
      if(client.subscribe(root_topic_subscribe)){
        Serial.println("Suscripcion a topic "+ String(root_topic_subscribe));
      }else{
        Serial.println("fallo Suscripciión a topic "+ String(root_topic_subscribe));
      }
    } else {
      Serial.print("falló conexión broker:( con error -> ");
      Serial.print(client.state());
      Serial.println(" Intentamos de nuevo en 5 segundos");
      delay(5000);
    }
  }
}


//*****************************
//***       CALLBACK        ***
//*****************************

void callback(char* topic, byte* payload, unsigned int length){
  String incoming = "";
  Serial.print("Mensaje recibido desde -> ");
  Serial.print(topic);
  Serial.println("");
  for (int i = 0; i < length; i++) {
    incoming += (char)payload[i];
  }
  incoming.trim();
  Serial.println("Mensaje -> " + incoming);

}


