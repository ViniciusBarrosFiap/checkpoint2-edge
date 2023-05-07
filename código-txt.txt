#include <LiquidCrystal.h>

// definindo as constantes
const int ldrPin = A1;  // pino do LDR
const int sensorTempPino = A2; // pino do Sensor Temperatura
const int SensorUmidPino = A0; // pino sensor de humidade
const int greenLedPin = 6; // pino do LED verde
const int yellowLedPin = 7; // pino do LED amarelo
const int redLedPin = 8; // pino do LED vermelho
const int buzzerPin = 9; // pino da buzina
const int threshold = 500; // valor limite para ativar o LED amarelo
const int buzzerDuration = 3000; // tempo de duração da buzina


bool buzzerAtivo = false; //definindo buzzer ativo como falso

int porcem = 0; //porcentagem da umidade


float rLDR;
float vOut;
const int rs = 12, en = 11, d4 = 5, d5 = 4, d6 = 3, d7 = 2;
LiquidCrystal lcd(rs, en, d4, d5, d6, d7);

void setup() {
  // definindo os pinos dos leds e da buzina como saídas
  pinMode(greenLedPin, OUTPUT);
  pinMode(yellowLedPin, OUTPUT);
  pinMode(redLedPin, OUTPUT);
  pinMode(buzzerPin, OUTPUT);

  // inicializando display 16x2
  lcd.begin(16, 2);

  // exibindo mensagem inicial
  lcd.print("Agnelo Vinhos");
  lcd.setCursor(0, 1);
  lcd.print("Monitoramento");
  delay(2000); // aguardando 2 segundos antes de iniciar a leitura dos sensores

}

void loop() {
  int ldrValue = analogRead(ldrPin); // lendo o valor do LDR

  // faz a leitura da tensão no Sensor de Temperatura
  int sensorTempTensao = analogRead(sensorTempPino);

  // converte a tensão lida
  float tensao = sensorTempTensao * 5;
  tensao /= 1024; // resultado da tensao

  // converte a tensão lida em Graus Celsius
  float temperaturaC = (tensao - 0.5) * 100;

  // converte a temperatura em Graus Celsius para Fahrenheit
  float temperaturaF = (temperaturaC * 9 / 5) + 32;

  // Faz a leitura da tensao no Sensor de Umidade 
  int SensorUmidTensao=analogRead(SensorUmidPino);
    
  // Converte a tensao lida em porcentagem
  float porcem=map(SensorUmidTensao,0,1023,0,100);
  
  //Calcula media de 5 leituras da temperatura
  int leiturasTemp = 0;
  int mediaTemp = 0;
  for (int i = 0; i < 5; i++) {
  	leiturasTemp += temperaturaC;
    delay(1000); //delay a cada leitura para maior precisão
  }
  mediaTemp = leiturasTemp / 5; //resultado média de 5 leituras da temperatura
  
  //calcula média de 5 leituras da umidade
  int leiturasUmi = 0;
  int mediaUmi = 0;
  for (int x = 0; x < 5; x++) {
  	leiturasUmi += porcem;
    delay(1000); //delay a cada leitura para maior precisão
  }
  mediaUmi = leiturasUmi / 5; //resultado média de 5 leituras da umidade
  
  
  
  // Estado da temperatura em graus celcius
  //temperatura BOA
  if (mediaTemp > 10 and mediaTemp < 15){
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Temperatura OK");
    lcd.setCursor(0, 1);
    lcd.print("Temp. = ");
    lcd.setCursor(8,1);
    lcd.print(mediaTemp);
    lcd.setCursor(10,1);
    lcd.print("C");
    //delay(5000);
  }
  //Temperatura ALTA acima de 15 graus
  else if (mediaTemp > 15) {
  	digitalWrite(greenLedPin, LOW);
    digitalWrite(yellowLedPin, HIGH);
    digitalWrite(redLedPin, LOW);
    
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Temp. ALTA");
    lcd.setCursor(0, 1);
    lcd.print("Temp. = ");
    lcd.setCursor(8,1);
    lcd.print(mediaTemp);
    lcd.setCursor(10,1);
    lcd.print("C");
    
   	// faz com que a buzina toque sem parar enquanto estiver acima de 15 graus
    if (!buzzerAtivo) {
      buzzerAtivo = true;
      while (temperaturaC > 15) {
        tone(buzzerPin, 1000);
        delay(buzzerDuration);
        noTone(buzzerPin);
        sensorTempTensao = analogRead(sensorTempPino);
    	tensao = sensorTempTensao * 5.0 / 1024.0;
    	temperaturaC = (tensao - 0.5) * 100.0;
      }
      buzzerAtivo = false;
    }
   	else {
    	digitalWrite(greenLedPin, HIGH);
    	digitalWrite(yellowLedPin, LOW);
    	digitalWrite(redLedPin, LOW);
    	noTone(buzzerPin);
    	buzzerAtivo = false;
  }
    
  }
  //temperatura baixa menos que 10 graus  
  else {
    digitalWrite(greenLedPin, LOW);
  	digitalWrite(yellowLedPin, LOW);
 	digitalWrite(redLedPin, HIGH);
    
  	lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Temp. BAIXA");
    lcd.setCursor(0, 1);
    lcd.print("Temp. = ");
    lcd.setCursor(8,1);
    lcd.print(mediaTemp);
    lcd.setCursor(10,1);
    lcd.print("C");
   	// faz com que a buzina toque sem parar enquanto estiver abaixo de 10 graus
    if (!buzzerAtivo) {
      buzzerAtivo = true;
      while (temperaturaC < 10) {
        tone(buzzerPin, 1000);
        delay(buzzerDuration);
        noTone(buzzerPin);
        sensorTempTensao = analogRead(sensorTempPino);
    	tensao = sensorTempTensao * 5.0 / 1024.0;
    	temperaturaC = (tensao - 0.5) * 100.0;
      }
      buzzerAtivo = false;
    }
   	else {
    	digitalWrite(greenLedPin, HIGH);
    	digitalWrite(yellowLedPin, LOW);
    	digitalWrite(redLedPin, LOW);
    	noTone(buzzerPin);
    	buzzerAtivo = false;
  }
  }
  
  
  //Atualiza estado da umidade
  if (porcem > 50 and porcem < 70) {
  	lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Umidade OK");
    lcd.setCursor(0, 1);
    lcd.print("Umidade =");
    lcd.setCursor(10,1);
    lcd.print(mediaUmi);
    lcd.setCursor(12,1);
    lcd.print("%");
    //delay(5000);
  }
  else if (porcem > 70) {
  	lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Umidade ALTA");
    lcd.setCursor(0, 1);
    lcd.print("Umidade =");
    lcd.setCursor(10,1);
    lcd.print(mediaUmi);
    lcd.setCursor(12,1);
    lcd.print("%");
    delay(5000);
  }
  
  else {
    digitalWrite(greenLedPin, LOW);
    digitalWrite(yellowLedPin, LOW);
    digitalWrite(redLedPin, HIGH);
    
  	lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Umidade BAIXA");
    lcd.setCursor(0, 1);
    lcd.print("Umidade =");
    lcd.setCursor(10,1);
    lcd.print(mediaUmi);
    lcd.setCursor(12,1);
    lcd.print("%");
    
    if (!buzzerAtivo) {
     buzzerAtivo = true;
      while (porcem < 50) {
        tone(buzzerPin, 1000);
        delay(buzzerDuration);
        noTone(buzzerPin);
        int SensorUmidTensao=analogRead(SensorUmidPino);
        porcem=map(SensorUmidTensao,0,1023,0,100);
        
      }
      buzzerAtivo = false;
    }
   	else {
    	digitalWrite(greenLedPin, HIGH);
    	digitalWrite(yellowLedPin, LOW);
    	digitalWrite(redLedPin, LOW);
    	noTone(buzzerPin);
    	buzzerAtivo = false;
  }
  }
  
  
  // verificando a luminosidade e acendendo o LED correspondente
  if (ldrValue > threshold) {
    digitalWrite(greenLedPin, HIGH);
    digitalWrite(yellowLedPin, LOW);
    digitalWrite(redLedPin, LOW);
  }
  else if (ldrValue > 260) {
    digitalWrite(greenLedPin, LOW);
    digitalWrite(yellowLedPin, HIGH);
    digitalWrite(redLedPin, LOW);
    
    lcd.clear();
  	lcd.setCursor(0, 0);
  	lcd.print("Ambiente a meia");
    lcd.setCursor(0, 2);
  	lcd.print("luz");
    
    // ativando a buzina por 3 segundos
    tone(buzzerPin, 1000);
    delay(buzzerDuration);
    noTone(buzzerPin);
  }
  else {
    digitalWrite(greenLedPin, LOW);
    digitalWrite(yellowLedPin, LOW);
    digitalWrite(redLedPin, HIGH);
    
    lcd.clear();
  	lcd.setCursor(0, 0);
  	lcd.print("Ambiente muito ");
    lcd.setCursor(0, 2);
  	lcd.print("CLARO");
   	
    if (!buzzerAtivo) {
      buzzerAtivo = true;
      while (ldrValue < 260) {
        tone(buzzerPin, 1000);
        delay(buzzerDuration);
        noTone(buzzerPin);
        ldrValue = analogRead(ldrPin);
      }
      buzzerAtivo = false;
    }
   	else {
    	digitalWrite(greenLedPin, HIGH);
    	digitalWrite(yellowLedPin, LOW);
    	digitalWrite(redLedPin, LOW);
    	noTone(buzzerPin);
    	buzzerAtivo = false;
  }
    
  }
}
