#include <LiquidCrystal.h>

// ===== PIN DEFINISI =====
#define GAS_PIN A0		// GAS SENSOR
#define TEMP_PIN A1     // TMP36
#define LED_PIN 13		// Lampu Indikator
#define BUZZER_PIN 8	// Alarm
#define RELAY_PIN 7		// Ke Transistor -> Relay -> Kipas

// LCD (RS, E, D4, D5, D6, D7)
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

// ===== VARIABEL =====
int nilaiGas = 0;
int batasGas = 320;

float suhuC = 0;
float batasSuhu = 100;

void setup() {
  pinMode(LED_PIN, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(RELAY_PIN, OUTPUT);

  digitalWrite(LED_PIN, LOW);
  digitalWrite(BUZZER_PIN, LOW);
  digitalWrite(RELAY_PIN, LOW);

  Serial.begin(9600);

  lcd.begin(16, 2);
  lcd.setCursor(0, 0);
  lcd.print("Deteksi Gas&Suhu");
  lcd.setCursor(0, 1);
  lcd.print("Status: Aman");
}

void loop() {
  // ===== BACA SENSOR =====
  nilaiGas = analogRead(GAS_PIN);

  int adcTemp = analogRead(TEMP_PIN);
  float voltage = adcTemp * 5.0 / 1023.0;
  float suhuC = (voltage - 0.5) * 100.0;  // TMP36

  // ===== STATUS LOGIKA =====
  bool bahayaGas  = nilaiGas > batasGas;
  bool bahayaSuhu = suhuC > batasSuhu;

  // ===== SERIAL =====
  Serial.print("Gas: ");
  Serial.print(nilaiGas);
  Serial.print(" | Suhu: ");
  Serial.print(suhuC);
  Serial.print(" C | Status: ");
  Serial.println((bahayaGas || bahayaSuhu) ? "BAHAYA" : "AMAN");

  // ===== LCD BARIS 1 =====
  lcd.setCursor(0, 0);
  lcd.print("G:");
  lcd.print(nilaiGas);
  lcd.print(" T:");
  lcd.print(suhuC);
  lcd.print("C  ");

  // ===== LOGIKA UTAMA =====
  if (bahayaGas || bahayaSuhu) {
    digitalWrite(LED_PIN, HIGH);
    digitalWrite(BUZZER_PIN, HIGH);
    digitalWrite(RELAY_PIN, HIGH);

    lcd.setCursor(0, 1);
    if (bahayaGas && bahayaSuhu) {
      lcd.print("Gas & Suhu!!! ");
    } else if (bahayaGas) {
      lcd.print("Gas BAHAYA   ");
    } else {
      lcd.print("Suhu BAHAYA  ");
    }
  } else {
    digitalWrite(LED_PIN, LOW);
    digitalWrite(BUZZER_PIN, LOW);
    digitalWrite(RELAY_PIN, LOW);

    lcd.setCursor(0, 1);
    lcd.print("Status: Aman ");
  }

  delay(500);
}
