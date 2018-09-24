#include <TimerOne.h>
#include <ArduinoJson.h>
#include <Adafruit_NeoPixel.h>

//#include <FastGPIO.h>
//#define Adafruit_NeoPixel_USE_FAST_GPIO
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

/*The message structure definition*/
typedef struct Messages {
  String msg;
  int    param; 
} Message;

/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

//Asignacion y creacion de variables


//Variables de inicializacion de los neo pixeles 
uint8_t  mode   = 0,        // Current animation effect
         offset = 0;
uint32_t color  = 0X00A4B3; // Starting color
uint32_t prevTime; 


//Variables de neopixeles LED
  int R=255;
  int G=255;
  int B=255;
  int n_neopixel=31;   // numero de neopixeles utilizados


//RGB para aro
  int Rw=255;
  int Gw=255;
  int Bw=255;
  int gama=255;
  int Rw2=255;          //para blink
  int Gw2=255;          //para blink
  int Bw2=255;          //para blink
  int gama2=255;        //para blink
  int n_pixel_aro=24;   // numero de pixeles del aro


//Variables de animacion      
  long int Vel_Luces=6500;     //Establece la velocidad de los LED, si el valor es mayor la velovidad sera mas lenta(se sube alrededor de los 1000)
  long int Vel_Luces_aro=250;   //Establece la velocidad de los LED, si el valor es mayor la velovidad sera mas lenta(se sube alrededor de los 1000)
  long int tiempo_reset=15000;   //establece el numero de ciclos para el reset 100 son aproximadamente 10 seg
  int dir=1;                    //direccion animacion de neopixel, varia entre 0 y 1
  int activar_blink=1;          //activa el blink de neopixel con 1 y lo desactiva con 0
  int iteraciones_blink=3;      //nuemro de veces que se encenderan las luces de neopixel
  int tiempo_blink=250;         //tiempo entre encendido y apagado de luces en mili segundos de neopixel
  int activa_animacion_aro=0;   //activa la animacion del aro con 1 y lo desactiva con 0
  int max_brillo_aro=255;       //maximo brillo del aro en su animacion de opacamiento de 0 a 255
  int min_brillo_aro=5;         //minimo brillo del aro en su animacion de opacamiento de 0 a 255

  
Adafruit_NeoPixel strip = Adafruit_NeoPixel(n_pixel_aro, 9, NEO_GRBW + NEO_KHZ800); //numero de pixeles del aro contaos desde 1, pin de control, tipo de led mas frecuencia de operacion
Adafruit_NeoPixel pixels = Adafruit_NeoPixel(n_neopixel, 10); //numero de neopixeles contandos desde 1, pin de control

//Variables del pulsador
  int encender = 0;
  int anterior = 0;
  int estado = 0;
  int PULSADOR = 8; // Pin digital para el pulsador
  int LED = 7; // Pin digital para el LED, controla del smart film


// Variables de control de estados  
  int claro=0;
  int opaco=0;
  char Byte_entrada;
  int control_serial=0;
  int control_cinta=0;
  int control_pulsador=0;
  long int contador_vel=-1; 
  long int contador_vel_aro=-1;
  long int contador_vel_reset=-1;
  long int contador_vel_reset2=-1;
  int cont_blink=0;
  int cont_intensidad_aro=255;

 

//Definicion de objetos animacion, el numero al que se igualan indica la poscicion de inicio del objeto
  int circulo1=0;
  int circulo2=1;
  int circulo3=2;
  int circulo4=3;
  int circulo5=4;  

  int circulo6=15;
  int circulo7=16;
  int circulo8=17;
  int circulo9=18;
  int circulo10=19;  

/////////////////////////////////////////////////////////////////////////////////////////////////////////////

void setup(void){

// Set initial serial boud rate with defaults.
  Serial.begin(9600);

// max milliseconds to wait for serial data when using Serial.readBytesUntil()
  Serial.setTimeout(1000);

    pinMode(PULSADOR,INPUT); // Pin digital del pulsador como entrada
    pinMode(LED,OUTPUT);     // Pin digital del LED como salida,PIN 13
    pinMode(13,OUTPUT);     // Pin digital del LED como salida,PIN 13
    digitalWrite(LED,LOW);   // LED apagado, Se inicializa la pantalla en opaco
    digitalWrite(13,LOW);   // LED apagado, Se inicializa la pantalla en opaco

//Inicializacion aro
    pixels.begin();                                                     //se inician los pixel del aro
    pixels.setBrightness(120);                                           // brillo de todos los neopixel de 0 a 120
    prevTime = millis();                                                // se inicializa reloj de neopixeles
    for(int j =0; j < n_neopixel; j++){pixels.setPixelColor(j,0,0,0);}  // Se inicializan los neopixel en 'off'
    pixels.show();                                                      // se actualizan neopixel

//Inicializacion neopixel
    strip.begin();                                                                     //se inician los neopixel
    strip.setBrightness(120);                                                           // brillo de todos los neopixel de 0 a 120
    for(int j =0; j < n_pixel_aro; j++){strip.setPixelColor(j, strip.Color(0,0,0,0));} // Se inicializan los pixel del aro en 'off'
    strip.show();                                                                      // se actualizan pixeles del aro
}



//////////////////////////////////////////////////////CODIGO PRINCIPAL////////////////////////////////////////
void loop(void){

//Boton
//Solo funciona si la tira LED esta en funcionamiento

//Boton fisico///////////////////////////////////////////////////////////////////////
  if(control_cinta==0){
   
    
    estado = digitalRead(PULSADOR);    // Guardamos el estado actual del pulsador  
     
    
    if(estado && anterior == 0){       // Comparamos el estado actual y el anterior del pulsador(se oprime el boton)
      encender = 1 - encender;
      control_cinta=1;  
      contador_vel=-1;
      contador_vel_aro=-1;
      
      sendButtonPressedMsg();

      
      delay(300);}                     // Evita los rebotes del pulsador.
     anterior = estado;                // Actualizamos el estado del pulsador.   
  }

//Boton virtual///////////////////////////////////////////////////////////////////////
  if(control_cinta==0){

// When serial has available data -> message is arriving
  while (Serial.available() > 0) {
    Message virtualbuton = readMessage();
    if (virtualbuton.msg == "Button:pressedUX") {

         estado=virtualbuton.param;

    if(estado && anterior == 0){       // Comparamos el estado actual y el anterior del pulsador(se oprime el boton)
      encender = 1 - encender;
      control_cinta=1;
      contador_vel=-1;
      contador_vel_aro=-1;}
      anterior = estado;               // Actualizamos el estado del pulsador.  
     } 
    
    else if (virtualbuton.msg == "other:message") {} 
    else { Serial.println("<unsupported message>");}
   }
  }

//Boton temporizado///////////////////////////////////////////////////////////////////////funciona si la cinta esta apagada

  
//Conadores ciclos de animacion///////////////////////////////////////////////////////////////////////

if(control_cinta==0){
contador_vel=contador_vel+1;
contador_vel_aro=contador_vel_aro+1;}



if(control_cinta==1){
  
contador_vel_reset=contador_vel_reset+1;

if(contador_vel_reset==5000){contador_vel_reset2=contador_vel_reset2+1;contador_vel_reset=-1;}

if(contador_vel_reset2==tiempo_reset){
digitalWrite(13,HIGH);
contador_vel_reset2=-1;

 Message m;
 m.msg ="glass:opacity";
 m.param=100;
    if (m.msg == "glass:opacity") {
      setGlassOpacity(m.param);///////////////////
    } else if (m.msg == "other:message") {
      // execute otherCommand(m.param)
    } else {
      Serial.println("<unsupported message>");
    }
}
}

/////////////////////////////////////////////////////////////////////////////////////////// secuencia apagado de cinta       

if(control_cinta==1){ 

if(activar_blink==0){ 
 for(int j =0; j < n_neopixel; j++){pixels.setPixelColor(j,0,0,0);}// se apagan neopixeles
 for(int j =0; j < n_pixel_aro; j++){strip.setPixelColor(j, strip.Color(0,0,0,0));}//se apagan pixels del aro
 pixels.show(); // se actualizan neopixel
 strip.show();  // se actualizan pixeles del aro
 }

if(activar_blink==1){ 
  
 if(cont_blink==0){ 
 for(int j =0; j < n_neopixel; j++){pixels.setPixelColor(j,0,0,0);}// se apagan neopixeles
 for(int j =0; j < n_pixel_aro; j++){strip.setPixelColor(j, strip.Color(0,0,0,0));}//se apagan pixels del aro
 pixels.show(); // se actualizan neopixel
 strip.show();  // se actualizan pixeles del aro
 digitalWrite(LED, LOW); // LED encendido, se aclara la pantalla

 delay(tiempo_blink);

for(int b=0;b<iteraciones_blink;b++){

 for(int j =0; j < n_neopixel; j++){pixels.setPixelColor(j,G,R,B);}// se encienden neopixeles
 for(int j =0; j < n_pixel_aro; j++){strip.setPixelColor(j, strip.Color(Rw2,Gw2,Bw2,gama2));}//se enciende pixels del aro
 pixels.show(); // se actualizan neopixel
 strip.show();  // se actualizan pixeles del aro

delay(tiempo_blink); 

 for(int j =0; j < n_neopixel; j++){pixels.setPixelColor(j,0,0,0);}// se apagan neopixeles
 for(int j =0; j < n_pixel_aro; j++){strip.setPixelColor(j, strip.Color(0,0,0,0));}//se apagan pixels del aro
 pixels.show(); // se actualizan neopixel
 strip.show();  // se actualizan pixeles del aro

delay(tiempo_blink);
cont_blink=1;}
 
}}
}

///////////////////////////////////////////////////////////////////////////////////////// Cinta activa 
if(contador_vel==Vel_Luces){
  
  if(control_cinta==0){

//construccion de objetos
    
  pixels.setPixelColor(circulo1,0,0,0);//g,r,b
  pixels.setPixelColor(circulo2,G,R,B);
  pixels.setPixelColor(circulo3,100,100,100);
  pixels.setPixelColor(circulo4,10,10,10);
  pixels.setPixelColor(circulo5,0,0,0);

  pixels.setPixelColor(circulo6,0,0,0);//g,r,b
  pixels.setPixelColor(circulo7,G,R,B);
  pixels.setPixelColor(circulo8,100,100,100);
  pixels.setPixelColor(circulo9,10,10,10);
  pixels.setPixelColor(circulo10,0,0,0);
 
//Corrimiento de los objetos   
 
 if(dir==0){
 circulo1=circulo1+1; 
 circulo2=circulo2+1; 
 circulo3=circulo3+1;
 circulo4=circulo4+1;
 circulo5=circulo5+1;
 
 circulo6=circulo6+1; 
 circulo7=circulo7+1; 
 circulo8=circulo8+1;
 circulo9=circulo9+1;
 circulo10=circulo10+1;}

  if(dir==1){
 circulo1=circulo1-1; 
 circulo2=circulo2-1; 
 circulo3=circulo3-1;
 circulo4=circulo4-1;
 circulo5=circulo5-1;

 circulo6=circulo6-1; 
 circulo7=circulo7-1; 
 circulo8=circulo8-1;
 circulo9=circulo9-1;
 circulo10=circulo10-1;}

//Retorno a posicion inicial de los objetos 
 if(dir==0){
 if(circulo1==n_neopixel){circulo1=0;}
 if(circulo2==n_neopixel){circulo2=0;}
 if(circulo3==n_neopixel){circulo3=0;}
 if(circulo4==n_neopixel){circulo4=0;}
 if(circulo5==n_neopixel){circulo5=0;}
 
 if(circulo6==n_neopixel){circulo6=0;}
 if(circulo7==n_neopixel){circulo7=0;}
 if(circulo8==n_neopixel){circulo8=0;}
 if(circulo9==n_neopixel){circulo9=0;}
 if(circulo10==n_neopixel){circulo10=0;}}

 if(dir==1){
 if(circulo1==-1){circulo1=(n_neopixel-1);}
 if(circulo2==-1){circulo2=(n_neopixel-1);}
 if(circulo3==-1){circulo3=(n_neopixel-1);}
 if(circulo4==-1){circulo4=(n_neopixel-1);}
 if(circulo5==-1){circulo5=(n_neopixel-1);}

 if(circulo6==-1){circulo6=(n_neopixel-1);}
 if(circulo7==-1){circulo7=(n_neopixel-1);}
 if(circulo8==-1){circulo8=(n_neopixel-1);}
 if(circulo9==-1){circulo9=(n_neopixel-1);}
 if(circulo10==-1){circulo10=(n_neopixel-1);}}
  
 }
pixels.show(); // se actualizan neopixel
contador_vel=-1;
}

///////////////////////////////////////////////////////////////////////////////////////// Aro activo 
if(contador_vel_aro==Vel_Luces_aro){
  
if(control_cinta==0){

if (activa_animacion_aro==0){for(int j =0; j < n_pixel_aro; j++){strip.setPixelColor(j, strip.Color(Rw,Gw,Bw,gama));}}

if (activa_animacion_aro==1){    
for(int j =0; j < n_pixel_aro; j++){strip.setPixelColor(j, strip.Color(Rw,Gw,Bw,gama));}

//animacion de opacamiento  
if (cont_intensidad_aro==255){
Rw=Rw-5;
Gw=Gw-5;
Bw=Bw-5;
gama=gama-5;
if(gama==min_brillo_aro){cont_intensidad_aro=0;}}

if (cont_intensidad_aro==0){
Rw=Rw+5;
Gw=Gw+5;
Bw=Bw+5;
gama=gama+5;
if(gama==max_brillo_aro){cont_intensidad_aro=255;}}

}}

strip.show();  // se actualizan pixeles del aro
contador_vel_aro=-1;
}

/////////////////////////////////////////////////////////////////////////////////////////mensajes y control de film

/*Si el estado del pulsador pasa de "LOW" a "HIGH"*/  
if(encender==1){ 

if(opaco==0){  
digitalWrite(LED, LOW); // LED apagado,se opaca la pantalla
opaco=1;}


//if(control_serial==0){control_serial=1;} 

// DSC button has been pressed! //Envia el mensaje por el puerto serial cuando el boton es pulsado
// When serial has available data -> message is arriving

  while (Serial.available() > 0) {
    Message m = readMessage();
    if (m.msg == "glass:opacity") {
      setGlassOpacity(m.param);///////////////////
    } else if (m.msg == "other:message") {
      // execute otherCommand(m.param)
    } else {
      Serial.println("<unsupported message>");}}

digitalWrite(LED, LOW); // LED apagado,se opaca la pantalla
claro=0;
    }
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////     
else{                                                      // Si el estado interno del pulsador pasa de "HIGH" a "LOW".  

if(claro==0){
digitalWrite(LED, HIGH); // LED encendido, se aclara la pantalla
claro=1;}

//control_serial=0;

digitalWrite(LED, HIGH); // LED encendido, se aclara la pantalla
opaco=0;
    }   
}//corchete del void
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

// =============================
//    DSC SwyftStore functions 
// =============================

/**
 * Sets the opacity of the glass door.
 * 
 *   opacity = 0   // Completely transparent
 *   opacity = 100 // Completely opaque
 */
void setGlassOpacity(int opacity) {
  if (opacity == 0) {          // DSC glass should be completely transparent
  } 
  
  else if (opacity == 100) { // DSC glass should be completely opaque
     digitalWrite(LED, LOW);   // LED apagado, Se inicializa la pantalla en opaco
     encender=0;       // variable de control
     cont_blink=0;     // variable de control
     control_cinta=0;  // variable de control
    // control_serial=0; // variable de control
     Rw=255;
     Gw=255;
     Bw=255;
     gama=255;
     cont_intensidad_aro=255;
     contador_vel=-1;
     contador_vel_aro=-1;
     digitalWrite(13,LOW);
  }
}

/**
 * Send a button pressed event.
 */
void sendButtonPressedMsg() {  
  Message evt = {"button:pressed", true};
  sendMessage(evt);
}

// =============================
//    Utility infra functions 
// =============================

/**
 * Reads a message encoded in json from the serial.
 */
Message readMessage() {
  StaticJsonBuffer<64> jsonBuffer;
  // Expects the Serial to have available() bytes.
  JsonObject& root = jsonBuffer.parseObject(Serial);

  if (!root.success()) {
    return (Message) {"<parse error>", -1};
  }

  const char* msgStr = root["msg"];
  String msg = String(msgStr);
  int param  = root["param"];

  return (Message) {msg, param};
}

/**
 * Sends the message encoding it as json through the Serial.
 */
void sendMessage(Message m) {
  String json = "{\"msg\": \"" + m.msg +"\", \"param\": " + m.param + "}";
  Serial.println(json);
}


