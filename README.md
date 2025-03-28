evaluatie week 7: Freestyle IoT project:

Voor dit project heb ik ervoor gekozen om een automatisch irrigatiesysteem te maken voor planten die ook de temperatuur en luchtvochtigheid van de kamer meten. Via blynk kan je aanpassen wat je gewenste bodemvochtigheid is.

Youtube link: https://youtube.com/shorts/SDGWMxC2Gw4?si=gg5hXB454pldegpX

componenten: 
-Grondvochtigheid sensor
-DHT11
-220ohm weerstand
-rode led (pompje werkte niet, zie video)
-ESP32

code:
#define BLYNK_TEMPLATE_ID "user7"
#define BLYNK_TEMPLATE_NAME "user7@server.wyns.it" // De server waarmee je verbonden bent
#define BLYNK_PRINT Serial

#include <WiFi.h>
#include <BlynkSimpleEsp32.h>
#include <DHT.h>

#define DHTPIN 23         // Pin voor de DHT11 sensor
#define DHTTYPE DHT11     // DHT11 type sensor

char auth[] = ""; // Vul je Blynk token in
char ssid[] = ""; // Vul je WiFi-SSID in
char pass[] = ""; // Vul je WiFi-wachtwoord in

const int soilMoisturePin = 34;  // Pin voor bodemvochtigheidssensor
const int ledPin = 13;           // Pin voor de LED

DHT dht(DHTPIN, DHTTYPE);

bool ledState = false; // Houd de status van de LED bij
int desiredMoisture = 50; // Drempelwaarde voor bodemvochtigheid die kan worden aangepast met de slider (standaard 50%)

BLYNK_WRITE(V4) { // De slider op Blynk stuurt de gewenste bodemvochtigheid
  desiredMoisture = param.asInt();  // Verkrijg de waarde van de slider (0-100%)
  Serial.print("Gewenste bodemvochtigheid: ");
  Serial.println(desiredMoisture);
}

void setup() {
  // Start seriële communicatie voor debuggen
  Serial.begin(115200);
  
  // Verbinden met Wi-Fi
  Serial.print("Connecting to WiFi...");
  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  Serial.println("Connected to WiFi!");

  // Blynk initialisatie voor lokale server
  Serial.println("Connecting to Blynk...");
  Blynk.begin(auth, ssid, pass, "server.wyns.it", 8081); // Verbind met lokale server op poort 8081
  
  // Initialiseer de DHT11 sensor
  dht.begin();
  
  // Stel de LED pin in als uitgang
  pinMode(ledPin, OUTPUT);
  
  // Zorg ervoor dat de LED uit is bij het opstarten
  digitalWrite(ledPin, LOW);
}

void loop() {
  // Lees de temperatuur en luchtvochtigheid van de DHT11
  float humidity = dht.readHumidity();
  float temperature = dht.readTemperature();
  
  // Lees de waarde van de bodemvochtigheidssensor
  int soilMoistureValue = analogRead(soilMoisturePin);
  
  // Zet de analoge waarde om naar percentage
  float soilMoisturePercent = map(soilMoistureValue, 0, 4095, 0, 100); // Zet de analoge waarde om naar een percentage (0-100%)
  
  // Print de sensorwaarden voor debuggen
  Serial.print("Temperatuur: ");
  Serial.print(temperature);
  Serial.print(" °C, Luchtvochtigheid: ");
  Serial.print(humidity);
  Serial.print(" %, Bodemvochtigheid: ");
  Serial.print(soilMoisturePercent);
  Serial.println(" %");
  
  // Stuur de gegevens naar Blynk
  Blynk.virtualWrite(V1, temperature);  // Temperatuur naar Blynk
  Blynk.virtualWrite(V2, humidity);     // Luchtvochtigheid naar Blynk
  Blynk.virtualWrite(V3, soilMoisturePercent); // Bodemvochtigheid naar Blynk (als percentage)
  
  // Als de bodem te droog is (waarde onder de drempel), zet de LED aan
  if (soilMoisturePercent < desiredMoisture) {  // Vergelijk met de drempelwaarde van de slider
    digitalWrite(ledPin, HIGH); // Zet de LED aan
    ledState = true; // LED is aan
  } else {
    digitalWrite(ledPin, LOW); // Zet de LED uit
    ledState = false; // LED is uit
  }

  // Stuur de LED-status naar Blynk (V5)
  Blynk.virtualWrite(V5, ledState);  // LED-status (1 = aan, 0 = uit) naar Virtual Pin V5

  // Blynk proces
  Blynk.run();
  
  delay(1000); // Wacht 1 seconde voordat de volgende meting wordt uitgevoerd
} 
