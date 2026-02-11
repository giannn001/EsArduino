int sensorPin = 2;
int TIP120pin = 9;
unsigned long start_time = 0;
unsigned long end_time = 0;
int motstate = 0;
int PWM;
int steps = 0;
float steps_old = 0;
float temp = 0;
float rps = 0;
float rpm = 0;
float impulsi_al_secondo = 0;
float tempo_per_giro = 0;

void setup() {
  Serial.begin(9600);
  pinMode(sensorPin, INPUT);
}

void loop() {
  PWM = analogRead(0);
  motstate = map(PWM, 0, 1023, 0, 255);
  
  if (motstate < 25){
    motstate = 0;
  }
  analogWrite(TIP120pin, motstate);
  
  start_time = millis();
  end_time = start_time + 1000;
  
  while (millis() < end_time)
  {
    if ((digitalRead(sensorPin)) == 1)
    {
      steps = steps + 1;
      while (digitalRead(sensorPin) == 1);
    }
  }
  
  temp = steps - steps_old;
  steps_old = steps;
  
  // Calcolo impulsi al secondo
  impulsi_al_secondo = temp;
  
  rps = (temp / 20);
  rpm = (rps * 60);
  
  // Calcolo tempo per giro completo (in secondi)
  if (rps > 0) {
    tempo_per_giro = 1.0 / rps;
  } else {
    tempo_per_giro = 0;
  }
  
  Serial.print("Impulsi/sec = ");
  Serial.print(impulsi_al_secondo, 1);
  Serial.print(", Tempo/giro = ");
  Serial.print(tempo_per_giro, 3);
  Serial.print(" s, rps = ");
  Serial.print(rps, 2);
  Serial.print(", rpm = ");
  Serial.println(rpm, 1);
  
  delay(1);
}
