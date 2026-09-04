#include <WiFi.h> 

#include <WebServer.h> 

#include <DHT.h> 

  

// ====================================================== 

// WIFI 

// ====================================================== 

  

const char* ssid = "AULASENSE"; 

const char* password = "CMSB2026+"; 

  

WebServer server(80); 

  

// ====================================================== 

// DHT11 

// ====================================================== 

  

#define DHTPIN 4 

#define DHTTYPE DHT11 

  

DHT dht(DHTPIN, DHTTYPE); 

  

// ====================================================== 

// PIR 

// ====================================================== 

  

const int pir = 27; 

bool pirTerminado = false; 

  

// ====================================================== 

// RELÉ / VENTILADORES 

// ====================================================== 

  

const int rele = 23; 

  

const int RELE_ON = HIGH; 

const int RELE_OFF = LOW; 

  

// ====================================================== 

// BUZZER 

// ====================================================== 

  

const int buzzer = 14; 

  

// ====================================================== 

// SENSOR MQ-2 

// ====================================================== 

  

const int sensorGas = 35; 

  

// ====================================================== 

// SENSOR LDR 

// ====================================================== 

  

const int sensorLuz = 32; 

  

// ====================================================== 

// MICRÓFONO 

// ====================================================== 

  

const int microfono = 34; 

  

// ====================================================== 

// RGB #1 = MICRÓFONO 

// ====================================================== 

  

const int rojo1 = 25; 

const int verde1 = 26; 

const int azul1 = 33; 

  

// ====================================================== 

// RGB #2 = LDR 

// ====================================================== 

  

const int rojo2 = 16; 

const int verde2 = 17; 

const int azul2 = 18; 

  

// ====================================================== 

// RGB #3 = TEMPERATURA 

// ====================================================== 

  

const int rojo3 = 19; 

const int verde3 = 21; 

const int azul3 = 22; 

  

// ====================================================== 

// UMBRALES 

// ====================================================== 

  

const float temperaturaAlta = 26.0; 

  

const int gasNormal = 1000; 

const int gasCritico = 1500; 

  

const int sonidoAlto = 1000; 

  

const int luzBaja = 57; 

  

// ====================================================== 

// VARIABLES DE SENSORES 

// ====================================================== 

  

float temperatura = 0; 

  

int gas = 0; 

int sonido = 0; 

int luz = 0; 

  

bool movimiento = false; 

  

bool alarmaCritica = false; 

  

// ====================================================== 

// MODOS MANUALES 

// ====================================================== 

// 

// 0 = AUTOMÁTICO 

// 1 = ENCENDIDO 

// 2 = APAGADO 

// 

  

int modoVentiladores = 0; 

int modoAlarma = 0; 

  

// ====================================================== 

// CONTROL DE TIEMPOS 

// ====================================================== 

  

unsigned long tiempoSensores = 0; 

  

const unsigned long intervaloSensores = 1000; 

  

// Mostrar IP cada 10 segundos 

unsigned long ultimoMensajeIP = 0; 

  

const unsigned long intervaloIP = 10000; 

  

// Parpadeo alarma 

unsigned long tiempoParpadeo = 0; 

  

bool estadoParpadeo = false; 

  

// ====================================================== 

// RGB #1 - MICRÓFONO 

// ====================================================== 

  

void apagarRGB1() { 

  

  digitalWrite(rojo1, LOW); 

  digitalWrite(verde1, LOW); 

  digitalWrite(azul1, LOW); 

  

} 

  

void rojo1Encendido() { 

  

  digitalWrite(rojo1, HIGH); 

  digitalWrite(verde1, LOW); 

  digitalWrite(azul1, LOW); 

  

} 

  

// ====================================================== 

// RGB #2 - LDR 

// ====================================================== 

  

void apagarRGB2() { 

  

  digitalWrite(rojo2, LOW); 

  digitalWrite(verde2, LOW); 

  digitalWrite(azul2, LOW); 

  

} 

  

void rojo2Encendido() { 

  

  digitalWrite(rojo2, HIGH); 

  digitalWrite(verde2, LOW); 

  digitalWrite(azul2, LOW); 

  

} 

  

void verde2Encendido() { 

  

  digitalWrite(rojo2, LOW); 

  digitalWrite(verde2, HIGH); 

  digitalWrite(azul2, LOW); 

  

} 

  

void azul2Encendido() { 

  

  digitalWrite(rojo2, LOW); 

  digitalWrite(verde2, LOW); 

  digitalWrite(azul2, HIGH); 

  

} 

  

// ====================================================== 

// RGB #3 - TEMPERATURA 

// ====================================================== 

  

void apagarRGB3() { 

  

  digitalWrite(rojo3, LOW); 

  digitalWrite(verde3, LOW); 

  digitalWrite(azul3, LOW); 

  

} 

  

void rojo3Encendido() { 

  

  digitalWrite(rojo3, HIGH); 

  digitalWrite(verde3, LOW); 

  digitalWrite(azul3, LOW); 

  

} 

  

// ====================================================== 

// VENTILADORES 

// ====================================================== 

  

void ventiladoresON() { 

  

  digitalWrite(rele, RELE_ON); 

  

} 

  

void ventiladoresOFF() { 

  

  digitalWrite(rele, RELE_OFF); 

  

} 

  

// ====================================================== 

// PÁGINA WEB 

// ====================================================== 

  

String paginaWeb() { 

  

  String html = R"rawliteral( 

  

<!DOCTYPE html> 

  

<html> 

  

<head> 

  

<meta charset="UTF-8"> 

  

<meta name="viewport" 

content="width=device-width, initial-scale=1.0"> 

  

<title>AULASENSE</title> 

  

<style> 

  

body { 

  

  font-family: Arial; 

  background: #111; 

  color: white; 

  margin: 0; 

  padding: 20px; 

  

} 

  

h1 { 

  

  text-align: center; 

  

} 

  

.contenedor { 

  

  display: grid; 

  

  grid-template-columns: 

  repeat(auto-fit, minmax(220px, 1fr)); 

  

  gap: 15px; 

  

  max-width: 1000px; 

  

  margin: auto; 

  

} 

  

.tarjeta { 

  

  background: #222; 

  

  padding: 20px; 

  

  border-radius: 15px; 

  

  text-align: center; 

  

} 

  

.valor { 

  

  font-size: 35px; 

  

  font-weight: bold; 

  

} 

  

.estado { 

  

  font-size: 20px; 

  

  margin-top: 10px; 

  

} 

  

button { 

  

  padding: 12px; 

  

  margin: 5px; 

  

  border: none; 

  

  border-radius: 8px; 

  

  cursor: pointer; 

  

  font-weight: bold; 

  

} 

  

.auto { 

  

  background: #3498db; 

  color: white; 

  

} 

  

.on { 

  

  background: #2ecc71; 

  color: white; 

  

} 

  

.off { 

  

  background: #e74c3c; 

  color: white; 

  

} 

  

.reset { 

  

  background: #f1c40f; 

  color: black; 

  

} 

  

canvas { 

  

  background: #181818; 

  border-radius: 10px; 

  width: 100%; 

  

} 

  

.critico { 

  

  background: #8b0000; 

  padding: 15px; 

  border-radius: 10px; 

  text-align: center; 

  font-size: 22px; 

  font-weight: bold; 

  

} 

  

.normal { 

  

  background: #145a32; 

  padding: 15px; 

  border-radius: 10px; 

  text-align: center; 

  font-size: 22px; 

  font-weight: bold; 

  

} 

  

</style> 

  

</head> 

  

<body> 

  

<h1>🏫 AULASENSE</h1> 

  

<div id="estadoGeneral" 

class="normal"> 

  

SISTEMA NORMAL 

  

</div> 

  

<br> 

  

<div class="contenedor"> 

  

<div class="tarjeta"> 

  

<h2>🌡️ Temperatura</h2> 

  

<div id="temperatura" 

class="valor"> 

  

-- 

  

</div> 

  

<p>°C</p> 

  

</div> 

  

<div class="tarjeta"> 

  

<h2>🔥 Gas MQ-2</h2> 

  

<div id="gas" 

class="valor"> 

  

-- 

  

</div> 

  

</div> 

  

<div class="tarjeta"> 

  

<h2>🎤 Micrófono</h2> 

  

<div id="sonido" 

class="valor"> 

  

-- 

  

</div> 

  

</div> 

  

<div class="tarjeta"> 

  

<h2>💡 Luz LDR</h2> 

  

<div id="luz" 

class="valor"> 

  

-- 

  

</div> 

  

</div> 

  

<div class="tarjeta"> 

  

<h2>🚶 PIR</h2> 

  

<div id="pir" 

class="estado"> 

  

-- 

  

</div> 

  

</div> 

  

<div class="tarjeta"> 

  

<h2>🌀 Ventiladores</h2> 

  

<div id="ventiladores" 

class="estado"> 

  

-- 

  

</div> 

  

</div> 

  

<div class="tarjeta"> 

  

<h2>🚨 Alarma</h2> 

  

<div id="alarma" 

class="estado"> 

  

-- 

  

</div> 

  

</div> 

  

</div> 

  

<br> 

  

<div class="tarjeta"> 

  

<h2>Control de ventiladores</h2> 

  

<button class="auto" 

onclick="ventiladores(0)"> 

  

AUTOMÁTICO 

  

</button> 

  

<button class="on" 

onclick="ventiladores(1)"> 

  

ENCENDER 

  

</button> 

  

<button class="off" 

onclick="ventiladores(2)"> 

  

APAGAR 

  

</button> 

  

</div> 

  

<br> 

  

<div class="tarjeta"> 

  

<h2>Control de alarma</h2> 

  

<button class="auto" 

onclick="alarma(0)"> 

  

AUTOMÁTICO 

  

</button> 

  

<button class="on" 

onclick="alarma(1)"> 

  

ENCENDER 

  

</button> 

  

<button class="off" 

onclick="alarma(2)"> 

  

APAGAR 

  

</button> 

  

</div> 

  

<br> 

  

<div class="tarjeta"> 

  

<h2>⚙️ Sistema</h2> 

  

<button class="reset" 

onclick="automatico()"> 

  

REINICIAR AUTOMÁTICO 

  

</button> 

  

</div> 

  

<br> 

  

<div class="tarjeta"> 

  

<h2>📈 Temperatura</h2> 

  

<canvas 

id="graficaTemperatura" 

width="600" 

height="250"> 

  

</canvas> 

  

</div> 

  

<br> 

  

<div class="tarjeta"> 

  

<h2>📈 Gas</h2> 

  

<canvas 

id="graficaGas" 

width="600" 

height="250"> 

  

</canvas> 

  

</div> 

  

<br> 

  

<div class="tarjeta"> 

  

<h2>📋 Límites del sistema</h2> 

  

<p> 

Temperatura alta: 

<b>Mayor a 26 °C</b> 

</p> 

  

<p> 

Gas normal: 

<b>Menor a 1000</b> 

</p> 

  

<p> 

Gas crítico: 

<b>1500 o más</b> 

</p> 

  

<p> 

Luz baja: 

<b>Mayor a 57</b> 

</p> 

  

<p> 

Sonido alto: 

<b>1000 o más</b> 

</p> 

  

</div> 

  

<script> 

  

let temperaturas = []; 

  

let gases = []; 

  

// ==================================================== 

// ACTUALIZAR DATOS 

// ==================================================== 

  

function actualizar() { 

  

fetch('/datos') 

  

.then(response => response.json()) 

  

.then(data => { 

  

document.getElementById( 

"temperatura" 

).innerHTML = 

data.temperatura.toFixed(1); 

  

document.getElementById( 

"gas" 

).innerHTML = 

data.gas; 

  

document.getElementById( 

"sonido" 

).innerHTML = 

data.sonido; 

  

document.getElementById( 

"luz" 

).innerHTML = 

data.luz; 

  

document.getElementById( 

"pir" 

).innerHTML = 

data.pir 

? "MOVIMIENTO" 

: "SIN MOVIMIENTO"; 

  

document.getElementById( 

"ventiladores" 

).innerHTML = 

data.ventiladores 

? "ENCENDIDOS" 

: "APAGADOS"; 

  

document.getElementById( 

"alarma" 

).innerHTML = 

data.alarma 

? "ACTIVA" 

: "APAGADA"; 

  

let estado = 

document.getElementById( 

"estadoGeneral" 

); 

  

if (data.critico) { 

  

estado.className = 

"critico"; 

  

estado.innerHTML = 

"🚨 ALARMA CRÍTICA"; 

  

} 

  

else { 

  

estado.className = 

"normal"; 

  

estado.innerHTML = 

"SISTEMA NORMAL"; 

  

} 

  

temperaturas.push( 

data.temperatura 

); 

  

gases.push( 

data.gas 

); 

  

if ( 

temperaturas.length > 40 

) { 

  

temperaturas.shift(); 

  

} 

  

if ( 

gases.length > 40 

) { 

  

gases.shift(); 

  

} 

  

dibujarGrafica( 

"graficaTemperatura", 

temperaturas 

); 

  

dibujarGrafica( 

"graficaGas", 

gases 

); 

  

}) 

  

.catch(error => { 

  

console.log( 

"Error obteniendo datos:", 

error 

); 

  

}); 

  

} 

  

// ==================================================== 

// GRÁFICAS 

// ==================================================== 

  

function dibujarGrafica( 

id, 

valores 

) { 

  

let canvas = 

document.getElementById(id); 

  

let ctx = 

canvas.getContext("2d"); 

  

ctx.clearRect( 

0, 

0, 

canvas.width, 

canvas.height 

); 

  

if ( 

valores.length < 2 

) { 

  

return; 

  

} 

  

let max = 

Math.max(...valores); 

  

let min = 

Math.min(...valores); 

  

if (max == min) { 

  

max++; 

min--; 

  

} 

  

ctx.beginPath(); 

  

for ( 

let i = 0; 

i < valores.length; 

i++ 

) { 

  

let x = 

i * 

( 

canvas.width / 

(valores.length - 1) 

); 

  

let y = 

canvas.height - 

( 

(valores[i] - min) / 

(max - min) 

) * 

canvas.height; 

  

if (i == 0) { 

  

ctx.moveTo( 

x, 

y 

); 

  

} 

  

else { 

  

ctx.lineTo( 

x, 

y 

); 

  

} 

  

} 

  

ctx.stroke(); 

  

} 

  

// ==================================================== 

// CONTROL VENTILADORES 

// ==================================================== 

  

function ventiladores(modo) { 

  

fetch( 

'/ventiladores?modo=' 

+ modo 

); 

  

} 

  

// ==================================================== 

// CONTROL ALARMA 

// ==================================================== 

  

function alarma(modo) { 

  

fetch( 

'/alarma?modo=' 

+ modo 

); 

  

} 

  

// ==================================================== 

// AUTOMÁTICO 

// ==================================================== 

  

function automatico() { 

  

fetch( 

'/automatico' 

); 

  

} 

  

// Actualizar cada segundo 

  

setInterval( 

actualizar, 

1000 

); 

  

actualizar(); 

  

</script> 

  

</body> 

  

</html> 

  

)rawliteral"; 

  

  return html; 

  

} 

  

// ====================================================== 

// PÁGINA PRINCIPAL 

// ====================================================== 

  

void handleRoot() { 

  

  server.send( 

    200, 

    "text/html", 

    paginaWeb() 

  ); 

  

} 

  

// ====================================================== 

// DATOS PARA DASHBOARD 

// ====================================================== 

  

void handleDatos() { 

  

  bool ventiladoresEstado = 

    digitalRead(rele) 

    == RELE_ON; 

  

  bool alarmaEstado = 

    digitalRead(buzzer) 

    == HIGH; 

  

  String json = "{"; 

  

  json += "\"temperatura\":"; 

  json += String( 

    temperatura, 

    1 

  ); 

  

  json += ",\"gas\":"; 

  json += String( 

    gas 

  ); 

  

  json += ",\"sonido\":"; 

  json += String( 

    sonido 

  ); 

  

  json += ",\"luz\":"; 

  json += String( 

    luz 

  ); 

  

  json += ",\"pir\":"; 

  

  json += String( 

    movimiento 

    ? "true" 

    : "false" 

  ); 

  

  json += ",\"ventiladores\":"; 

  

  json += String( 

    ventiladoresEstado 

    ? "true" 

    : "false" 

  ); 

  

  json += ",\"alarma\":"; 

  

  json += String( 

    alarmaEstado 

    ? "true" 

    : "false" 

  ); 

  

  json += ",\"critico\":"; 

  

  json += String( 

    alarmaCritica 

    ? "true" 

    : "false" 

  ); 

  

  json += "}"; 

  

  server.send( 

    200, 

    "application/json", 

    json 

  ); 

  

} 

  

// ====================================================== 

// CONTROL DE VENTILADORES 

// ====================================================== 

  

void handleVentiladores() { 

  

  if ( 

    !server.hasArg("modo") 

  ) { 

  

    server.send( 

      400, 

      "text/plain", 

      "Falta modo" 

    ); 

  

    return; 

  

  } 

  

  modoVentiladores = 

    server.arg( 

      "modo" 

    ).toInt(); 

  

  server.send( 

    200, 

    "text/plain", 

    "OK" 

  ); 

  

} 

  

// ====================================================== 

// CONTROL DE ALARMA 

// ====================================================== 

  

void handleAlarma() { 

  

  if ( 

    !server.hasArg("modo") 

  ) { 

  

    server.send( 

      400, 

      "text/plain", 

      "Falta modo" 

    ); 

  

    return; 

  

  } 

  

  modoAlarma = 

    server.arg( 

      "modo" 

    ).toInt(); 

  

  server.send( 

    200, 

    "text/plain", 

    "OK" 

  ); 

  

} 

  

// ====================================================== 

// VOLVER A AUTOMÁTICO 

// ====================================================== 

  

void handleAutomatico() { 

  

  modoVentiladores = 0; 

  

  modoAlarma = 0; 

  

  server.send( 

    200, 

    "text/plain", 

    "Sistema automático" 

  ); 

  

} 

  

// ====================================================== 

// SETUP 

// ====================================================== 

  

void setup() { 

  

  Serial.begin(115200); 

  

  // ==================================================== 

  // PINES 

  // ==================================================== 

  

  pinMode( 

    pir, 

    INPUT 

  ); 

  

  pinMode( 

    rele, 

    OUTPUT 

  ); 

  

  pinMode( 

    buzzer, 

    OUTPUT 

  ); 

  

  pinMode( 

    rojo1, 

    OUTPUT 

  ); 

  

  pinMode( 

    verde1, 

    OUTPUT 

  ); 

  

  pinMode( 

    azul1, 

    OUTPUT 

  ); 

  

  pinMode( 

    rojo2, 

    OUTPUT 

  ); 

  

  pinMode( 

    verde2, 

    OUTPUT 

  ); 

  

  pinMode( 

    azul2, 

    OUTPUT 

  ); 

  

  pinMode( 

    rojo3, 

    OUTPUT 

  ); 

  

  pinMode( 

    verde3, 

    OUTPUT 

  ); 

  

  pinMode( 

    azul3, 

    OUTPUT 

  ); 

  

  // ==================================================== 

  // ESTADOS INICIALES 

  // ==================================================== 

  

  ventiladoresOFF(); 

  

  digitalWrite( 

    buzzer, 

    LOW 

  ); 

  

  apagarRGB1(); 

  

  apagarRGB2(); 

  

  apagarRGB3(); 

  

  // ==================================================== 

  // DHT 

  // ==================================================== 

  

  dht.begin(); 

  

  // ==================================================== 

  // WIFI 

  // ==================================================== 

  

  WiFi.mode( 

    WIFI_STA 

  ); 

  

  WiFi.begin( 

    ssid, 

    password 

  ); 

  

  Serial.println(); 

  

  Serial.println( 

    "==============================" 

  ); 

  

  Serial.println( 

    "         AULASENSE" 

  ); 

  

  Serial.println( 

    "==============================" 

  ); 

  

  Serial.print( 

    "Conectando a: " 

  ); 

  

  Serial.println( 

    ssid 

  ); 

  

  while ( 

    WiFi.status() 

    != WL_CONNECTED 

  ) { 

  

    delay(500); 

  

    Serial.print( 

      "." 

    ); 

  

  } 

  

  Serial.println(); 

  

  Serial.println( 

    "==============================" 

  ); 

  

  Serial.println( 

    "       WIFI CONECTADO" 

  ); 

  

  Serial.println( 

    "==============================" 

  ); 

  

  Serial.print( 

    "Red: " 

  ); 

  

  Serial.println( 

    ssid 

  ); 

  

  Serial.print( 

    "IP DEL ESP32: " 

  ); 

  

  Serial.println( 

    WiFi.localIP() 

  ); 

  

  Serial.print( 

    "Dashboard: http://" 

  ); 

  

  Serial.println( 

    WiFi.localIP() 

  ); 

  

  Serial.println( 

    "==============================" 

  ); 

  

  // ==================================================== 

  // SERVIDOR WEB 

  // ==================================================== 

  

  server.on( 

    "/", 

    handleRoot 

  ); 

  

  server.on( 

    "/datos", 

    handleDatos 

  ); 

  

  server.on( 

    "/ventiladores", 

    handleVentiladores 

  ); 

  

  server.on( 

    "/alarma", 

    handleAlarma 

  ); 

  

  server.on( 

    "/automatico", 

    handleAutomatico 

  ); 

  

  server.begin(); 

  

  Serial.println( 

    "Servidor web iniciado" 

  ); 

  

} 

  

// ====================================================== 

// LOOP 

// ====================================================== 

  

void loop() { 

  

  // ==================================================== 

  // SERVIDOR WEB 

  // ==================================================== 

  

  server.handleClient(); 

  

  // ==================================================== 

  // MOSTRAR IP CADA 10 SEGUNDOS 

  // ==================================================== 

  

  if ( 

    millis() - ultimoMensajeIP 

    >= intervaloIP 

  ) { 

  

    ultimoMensajeIP = 

      millis(); 

  

    Serial.println(); 

  

    Serial.println( 

      "==============================" 

    ); 

  

    if ( 

      WiFi.status() 

      == WL_CONNECTED 

    ) { 

  

      Serial.print( 

        "IP DEL ESP32: " 

      ); 

  

      Serial.println( 

        WiFi.localIP() 

      ); 

  

      Serial.print( 

        "Dashboard: http://" 

      ); 

  

      Serial.println( 

        WiFi.localIP() 

      ); 

  

    } 

  

    else { 

  

      Serial.println( 

        "WIFI DESCONECTADO" 

      ); 

  

    } 

  

    Serial.println( 

      "==============================" 

    ); 

  

  } 

  

  // ==================================================== 

  // PIR SOLO AL INICIO 

  // ==================================================== 

  

  if (!pirTerminado) { 

  

    movimiento = 

      digitalRead( 

        pir 

      ); 

  

    if (movimiento) { 

  

      Serial.println( 

        "PIR DETECTADO" 

      ); 

  

      Serial.println( 

        "Ventiladores activados" 

      ); 

  

      ventiladoresON(); 

  

      pirTerminado = 

        true; 

  

      delay(1000); 

  

    } 

  

    return; 

  

  } 

  

  // ==================================================== 

  // LECTURA DE SENSORES 

  // ==================================================== 

  

  if ( 

    millis() - tiempoSensores 

    >= intervaloSensores 

  ) { 

  

    tiempoSensores = 

      millis(); 

  

    // -------------------------------------------------- 

    // DHT11 

    // -------------------------------------------------- 

  

    temperatura = 

      dht.readTemperature(); 

  

    // -------------------------------------------------- 

    // MQ-2 

    // -------------------------------------------------- 

  

    gas = 

      analogRead( 

        sensorGas 

      ); 

  

    // -------------------------------------------------- 

    // MICRÓFONO 

    // -------------------------------------------------- 

  

    sonido = 

      analogRead( 

        microfono 

      ); 

  

    // -------------------------------------------------- 

    // LDR 

    // -------------------------------------------------- 

  

    luz = 

      analogRead( 

        sensorLuz 

      ); 

  

    // -------------------------------------------------- 

    // PIR 

    // -------------------------------------------------- 

  

    movimiento = 

      digitalRead( 

        pir 

      ); 

  

    // ================================================== 

    // COMPROBAR DHT 

    // ================================================== 

  

    if ( 

      isnan(temperatura) 

    ) { 

  

      Serial.println( 

        "ERROR: DHT11" 

      ); 

  

      return; 

  

    } 

  

    // ================================================== 

    // ALARMA CRÍTICA 

    // ================================================== 

  

    alarmaCritica = 

      ( 

        temperatura >= 27.0 && 

        gas >= gasCritico 

      ); 

  

    // ================================================== 

    // ALARMA CRÍTICA TIENE PRIORIDAD 

    // ================================================== 

  

    if ( 

      alarmaCritica 

    ) { 

  

      Serial.println(); 

      Serial.println( 

        "🚨 ALARMA CRÍTICA" 

      ); 

  

      Serial.print( 

        "Temperatura: " 

      ); 

  

      Serial.println( 

        temperatura 

      ); 

  

      Serial.print( 

        "Gas: " 

      ); 

  

      Serial.println( 

        gas 

      ); 

  

      // ---------------------------------------------- 

      // VENTILADORES OFF 

      // ---------------------------------------------- 

  

      ventiladoresOFF(); 

  

      // ---------------------------------------------- 

      // BUZZER ON 

      // ---------------------------------------------- 

  

      digitalWrite( 

        buzzer, 

        HIGH 

      ); 

  

      // ---------------------------------------------- 

      // PARPADEO 

      // ---------------------------------------------- 

  

      if ( 

        millis() - tiempoParpadeo 

        >= 500 

      ) { 

  

        tiempoParpadeo = 

          millis(); 

  

        estadoParpadeo = 

          !estadoParpadeo; 

  

      } 

  

      if ( 

        estadoParpadeo 

      ) { 

  

        rojo1Encendido(); 

  

        rojo2Encendido(); 

  

        rojo3Encendido(); 

  

      } 

  

      else { 

  

        apagarRGB1(); 

  

        apagarRGB2(); 

  

        apagarRGB3(); 

  

      } 

  

      return; 

  

    } 

  

    // ================================================== 

    // NO HAY ALARMA CRÍTICA 

    // ================================================== 

  

    // ================================================== 

    // BUZZER 

    // ================================================== 

  

    if ( 

      modoAlarma == 1 

    ) { 

  

      digitalWrite( 

        buzzer, 

        HIGH 

      ); 

  

    } 

  

    else { 

  

      digitalWrite( 

        buzzer, 

        LOW 

      ); 

  

    } 

  

    // ================================================== 

    // TEMPERATURA 

    // ================================================== 

  

    if ( 

      temperatura > temperaturaAlta 

    ) { 

  

      rojo3Encendido(); 

  

    } 

  

    else { 

  

      apagarRGB3(); 

  

    } 

  

    // ================================================== 

    // GAS 

    // ================================================== 

  

    bool gasAlto = 

      gas >= gasNormal; 

  

    // ================================================== 

    // VENTILADORES 

    // ================================================== 

  

    if ( 

      modoVentiladores == 1 

    ) { 

  

      ventiladoresON(); 

  

    } 

  

    else if ( 

      modoVentiladores == 2 

    ) { 

  

      ventiladoresOFF(); 

  

    } 

  

    else { 

  

      // AUTOMÁTICO 

  

      if ( 

        temperatura > temperaturaAlta || 

        gasAlto 

      ) { 

  

        ventiladoresON(); 

  

      } 

  

      else { 

  

        ventiladoresOFF(); 

  

      } 

  

    } 

  

    // ================================================== 

    // MICRÓFONO 

    // ================================================== 

    // 

    // El micrófono tiene menor prioridad. 

    // 

    // Si temperatura o gas están altos, 

    // el micrófono no controla su RGB. 

    // ================================================== 

  

    if ( 

      temperatura <= temperaturaAlta && 

      gas < gasNormal 

    ) { 

  

      if ( 

        sonido >= sonidoAlto 

      ) { 

  

        rojo1Encendido(); 

  

      } 

  

      else { 

  

        apagarRGB1(); 

  

      } 

  

    } 

  

    else { 

  

      apagarRGB1(); 

  

    } 

  

    // ================================================== 

    // LDR 

    // ================================================== 

  

    if ( 

      luz > luzBaja 

    ) { 

  

      azul2Encendido(); 

  

    } 

  

    else { 

  

      verde2Encendido(); 

  

    } 

  

    // ================================================== 

    // MONITOR SERIAL 

    // ================================================== 

  

    Serial.println( 

      "----------------------------" 

    ); 

  

    Serial.print( 

      "Temperatura: " 

    ); 

  

    Serial.print( 

      temperatura 

    ); 

  

    Serial.println( 

      " C" 

    ); 

  

    Serial.print( 

      "Gas: " 

    ); 

  

    Serial.println( 

      gas 

    ); 

  

    Serial.print( 

      "Sonido: " 

    ); 

  

    Serial.println( 

      sonido 

    ); 

  

    Serial.print( 

      "Luz: " 

    ); 

  

    Serial.println( 

      luz 

    ); 

  

    Serial.print( 

      "Ventiladores: " 

    ); 

  

    Serial.println( 

      digitalRead(rele) 

      == RELE_ON 

      ? "ENCENDIDOS" 

      : "APAGADOS" 

    ); 

  

    Serial.print( 

      "Alarma critica: " 

    ); 

  

    Serial.println( 

      alarmaCritica 

      ? "SI" 

      : "NO" 

    ); 

  

  } 

  

} 

  
