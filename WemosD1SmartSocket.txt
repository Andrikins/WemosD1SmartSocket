#include <ESP8266WiFi.h>
#include <WiFiClient.h>
#include <ESP8266WebServer.h>
#include <ESP8266mDNS.h>

const char* ssid = "SamsungS8";
const char* password = "dkve8129";

ESP8266WebServer server(80);

const int PIN = D1;

void handleRoot() {
  String message = "<html><head></head><body style='font-family: sans-serif; font-size: 12px'>Following functions are available:<br><br>";
  message += "<a href='/ON'>Socket On</a> Socket ON<br>";
  message += "<a href='/OFF'>Socket OFF</a> Socket OFF<br>";
  server.send(200, "text/html", message);
}

void handleNotFound(){
  
  String message = "File Not Found\n\n";
  
  message += "URI: ";
  message += server.uri();
  message += "\nMethod: ";
  message += (server.method() == HTTP_GET)?"GET":"POST";
  message += "\nArguments: ";
  message += server.args();
  message += "\n";
  for (uint8_t i=0; i<server.args(); i++){
    message += " " + server.argName(i) + ": " + server.arg(i) + "\n";
  }
  server.send(404, "text/plain", message);
  
}

void setup(void){
  pinMode(PIN, OUTPUT);
  
  Serial.begin(115200);
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  Serial.println("");

  // Wait for connection
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.print("Connected to ");
  Serial.println(ssid);
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());

  if (MDNS.begin("esp8266")) {
    Serial.println("MDNS responder started");
  }

  server.on("/", handleRoot);

  server.on("/ON", [](){
    server.send(200, "text/plain", "this works as well");
    digitalWrite(PIN, LOW);
  });

  server.on("/OFF", [](){
    server.send(200, "text/plain", "this works as well");
    digitalWrite(PIN, HIGH);
  });
  server.onNotFound(handleNotFound);

  server.begin();
  Serial.println("HTTP server started");
}

void loop(void){
  server.handleClient();
}
