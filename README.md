#include <Adafruit_Sensor.h>
#include <DHT.h>
#include <DHT_U.h>
#include "LiquidCrystal.h"
#include <IRremote.h>

#define IR_PIN 11 // Pin connecté au récepteur infrarouge

IRrecv irReceiver(IR_PIN);
decode_results irResults;

// Broches pour le LCD
#define LCD_RS 8
#define LCD_EN 7
#define LCD_D4 6
#define LCD_D5 5
#define LCD_D6 4
#define LCD_D7 3

LiquidCrystal lcd(LCD_RS, LCD_EN, LCD_D4, LCD_D5, LCD_D6, LCD_D7);

// Broches pour le DHT11
#define DHTPIN 12
// Broches pour le buzzer et la lampe rouge
#define BUZZER_PIN 13
#define LED_PIN 14

// Décommentez le type de capteur en cours d'utilisation :
#define DHTTYPE    DHT11

DHT_Unified dht(DHTPIN, DHTTYPE);

uint32_t delayMS;

void setup() {
  Serial.begin(9600);
  irReceiver.enableIRIn(); // Activez la réception infrarouge

  // Initialiser le capteur.
  dht.begin();

  // Configurer les broches pour le buzzer et la lampe rouge
  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(LED_PIN, OUTPUT);

  // Initialiser l'écran LCD
  lcd.begin(16, 2); // Initialise un écran LCD 16x2

  Serial.println(F("DHTxx Unified Sensor Example"));
  sensor_t sensor;
  // Définir les détails du capteur de température.
  dht.temperature().getSensor(&sensor);
  Serial.println(F("------------------------------------"));
  Serial.println(F("Temperature Sensor"));
  Serial.print  (F("Sensor Type: ")); Serial.println(sensor.name);
  Serial.print  (F("Driver Ver:  ")); Serial.println(sensor.version);
  Serial.print  (F("Unique ID:   ")); Serial.println(sensor.sensor_id);
  Serial.print  (F("Max Value:   ")); Serial.print(sensor.max_value); Serial.println(F("°C"));
  Serial.print  (F("Min Value:   ")); Serial.print(sensor.min_value); Serial.println(F("°C"));
  Serial.print  (F("Resolution:  ")); Serial.print(sensor.resolution); Serial.println(F("°C"));
  Serial.println(F("------------------------------------"));
  // Définir les détails du capteur d'humidité.
  dht.humidity().getSensor(&sensor);
  Serial.println(F("Humidity Sensor"));
  Serial.print  (F("Sensor Type: ")); Serial.println(sensor.name);
  Serial.print  (F("Driver Ver:  ")); Serial.println(sensor.version);
  Serial.print  (F("Unique ID:   ")); Serial.println(sensor.sensor_id);
  Serial.print  (F("Max Value:   ")); Serial.print(sensor.max_value); Serial.println(F("%"));
  Serial.print  (F("Min Value:   ")); Serial.print(sensor.min_value); Serial.println(F("%"));
  Serial.print  (F("Resolution:  ")); Serial.print(sensor.resolution); Serial.println(F("%"));
  Serial.println(F("------------------------------------"));
  // Définir le délai entre les lectures du capteur en fonction des détails du capteur.
  delayMS = sensor.min_delay / 1000;
}

void loop() {
  if (irReceiver.decode(&irResults)) {
    // Si un signal infrarouge est reçu
    switch (irResults.value) {
      case 0xFF6897: // Code pour le bouton 1
        afficherTemperature();
        break;
      case 0xFF30CF: // Code pour le bouton 2
        afficherHumidite();
        break;
      case 0xFF18E7: // Code pour le bouton 3
        afficherHeure();
        break;
      case 0xFF7A85: // Code pour le bouton 4
        afficherTempHum();
        break;
      // Ajoutez d'autres cas pour d'autres boutons si nécessaire
    }
    irReceiver.resume(); // Attendre le prochain signal infrarouge
  }
  
  // Lecture de la température et de l'humidité
  delay(delayMS);
  sensors_event_t event;
  dht.temperature().getEvent(&event);

  if (isnan(event.temperature)) {
    Serial.println(F("Erreur de lecture de la température !"));
  } else {
    Serial.print(F("Température : "));
    Serial.print(event.temperature);
    Serial.println(F("°C"));

    // Gestion de la température
    if (event.temperature > 25.0) {
      // Température élevée : allumer la lampe rouge et activer le buzzer
      digitalWrite(LED_PIN, HIGH);
      tone(BUZZER_PIN, 1000);  // Émettre un son à 1000 Hz
      delay(500);  // Durée pendant laquelle le buzzer est actif (en millisecondes)
      noTone(BUZZER_PIN);  // Arrêter le buzzer
    } else {
      // Température normale : éteindre la lampe rouge et le buzzer
      digitalWrite(LED_PIN, LOW);
      noTone(BUZZER_PIN);
    }
  }

  dht.humidity().getEvent(&event);

  if (isnan(event.relative_humidity)) {
    Serial.println(F("Erreur de lecture de l'humidité !"));
  } else {
    Serial.print(F("Humidité : "));
    Serial.print(event.relative_humidity);
    Serial.println(F("%"));

    // Gestion de l'humidité
    if (event.relative_humidity > 70.0) {
      // Humidité élevée : allumer la lampe rouge et activer le buzzer
      digitalWrite(LED_PIN, HIGH);
      tone(BUZZER_PIN, 1000);
      delay(500);
      noTone(BUZZER_PIN);
    } else {
      // Humidité normale : éteindre la lampe rouge et le buzzer
      digitalWrite(LED_PIN, LOW);
      noTone(BUZZER_PIN);
    }
  }
}

void afficherTemperature() {
  // Code pour afficher la température sur le LCD ou dans la console série
}

void afficherHumidite() {
  // Code pour afficher l'humidité sur le LCD ou dans la console série
}

void afficherHeure() {
  // Code pour afficher l'heure sur le LCD ou dans la console série
}

void afficherTempHum() {
  // Code pour afficher la température et l'humidité sur le LCD ou dans la console série
}
