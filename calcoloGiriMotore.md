int sensorPin = 2;
int TIP120pin = 9;
unsigned long start_time = 0;
unsigned long end_time = 0;
unsigned long revolution_start_time = 0;
unsigned long revolution_time = 0;
int motstate = 0;
int PWM;
int steps = 0;
float steps_old = 0;
float temp = 0;
float rps = 0;
float rpm = 0;
int pulses_per_second = 0;
bool revolution_started = false;
int revolution_pulse_count = 0;

void setup() {
  Serial.begin(9600);
  pinMode(sensorPin, INPUT);
}

void loop() {
  PWM = analogRead(0);
  motstate = map(PWM, 0, 1023, 0, 255);
  
  if (motstate < 25) {
    motstate = 0;
  }
  
  analogWrite(TIP120pin, motstate);
  
  start_time = millis();
  end_time = start_time + 1000;
  pulses_per_second = 0;  // reset contatore impulsi al secondo

  while (millis() < end_time) {
    if ((digitalRead(sensorPin)) == 1) {
      steps = steps + 1;
      pulses_per_second = pulses_per_second + 1;
      
      // Inizia a contare il tempo per un giro completo
      if (!revolution_started) {
        revolution_start_time = millis();
        revolution_started = true;
        revolution_pulse_count = 0;
      }
      
      revolution_pulse_count++;
      
      // Se abbiamo completato 20 impulsi (un giro completo)
      if (revolution_pulse_count >= 20) {
        revolution_time = millis() - revolution_start_time;
        revolution_started = false;  // reset per il prossimo giro
      }
      
      while (digitalRead(sensorPin) == 1);  // attendi il rilascio
    }
  }

  temp = steps - steps_old;
  steps_old = steps;
  rps = (temp / 20);
  rpm = (rps * 60);

  // Stampa sul Serial Monitor
  Serial.print("Impulsi/secondo = ");
  Serial.print(pulses_per_second);
  Serial.print(", rps = ");
  Serial.print(rps);
  Serial.print(", rpm = ");
  Serial.print(rpm);
  
  if (revolution_time > 0) {
    Serial.print(", Tempo per giro = ");
    Serial.print(revolution_time);
    Serial.print(" ms (");
    Serial.print(revolution_time / 1000.0);
    Serial.print(" secondi)");
  }
  
  Serial.println();

  delay(1);
}
