# PRACTICA 7: BUSOS DE COMUNICACIÓ 3 (I2S)
### Autor: Tomàs Lloret

L'objectiu d'aquesta pràctica és describir i entendre el funcionament del bus I2S.

## Part A: Reproducció d'àudio desde la memòria interna
Iniciem el codi incloent les llibreries d'àudio necessàries per a realitzar aquesta pràctica correctament i definim tres punters, un que llegirà l'arxiu d'àudio, un segon que el decodificarà i un tercer que l'enviarà de forma analògica a l'output.
```c
#include "AudioGeneratorAAC.h"
#include "AudioOutputI2S.h"
#include "AudioFileSourcePROGMEM.h"
#include "sampleaac.h"

AudioFileSourcePROGMEM *in;
AudioGeneratorAAC *aac;
AudioOutputI2S *out;
```

Dins del setup, després de posar la velocitat de comunicació usual, inicialitzem un punter "in" amb la mostra d'el arxiu d'àudio .aac i la mida d'aquest com a paràmetre. Després d'aquest, farem un altre punter que codificarà dins del .aac i un tercer que serà la sortida d'àudio. Per acabar, s'inicialitzarà aquest tercer punter amb un guany de 0.125 i se li definiràn els pins de sortida.
```c
void setup(){
  Serial.begin(115200);

  in = new AudioFileSourcePROGMEM(sampleaac, sizeof(sampleaac));
  aac = new AudioGeneratorAAC();
  out = new AudioOutputI2S();
  out -> SetGain(0.125);
  out -> SetPinout(26,25,22);
  aac->begin(in, out);
}
```

En el loop primerament comprobarem que el fitxer .aac s'executi fins a estar completament decodificat. Quan acabi de decodificar-se, el procés acabarà i s'imprimirà "Sound Generator" per terminal.
```c
void loop(){
  if (aac->isRunning()) {
    aac->loop();
  } else {
    aac -> stop();
    Serial.printf("Sound Generator\n");
    delay(1000);
  }
}
```

### Part A: Codi complet
```c
#include "AudioGeneratorAAC.h"
#include "AudioOutputI2S.h"
#include "AudioFileSourcePROGMEM.h"
#include "sampleaac.h"

AudioFileSourcePROGMEM *in;
AudioGeneratorAAC *aac;
AudioOutputI2S *out;

void setup(){
  Serial.begin(115200);

  in = new AudioFileSourcePROGMEM(sampleaac, sizeof(sampleaac));
  aac = new AudioGeneratorAAC();
  out = new AudioOutputI2S();
  out -> SetGain(0.125);
  out -> SetPinout(26,25,22);
  aac->begin(in, out);
}

void loop(){
  if (aac->isRunning()) {
    aac->loop();
  } else {
    aac -> stop();
    Serial.printf("Sound Generator\n");
    delay(1000);
  }
}
```

## Part B: Reproducció d'àudio desde una tarjeta SD
Començem la part B de la pràctica inclloent les llibreries necessàries per al funcionament d'aquesta, SD.h serveix per a poder utilitzar una targeta SD i FS.h és una llibreria de control d'àudio. Després, definim els pins que utilitzarem: Els primers quatre s'usen per a connectar la nostre ESP32 al lector SD, i els tres següents ens connectaran al DAC MAX9835. Finalment, definim un objecte audio.
```c
#include "Audio.h"
#include "SD.h"
#include "FS.h"

// Digital I/O used, Tarjeta SD
#define SD_CS          5
#define SPI_MOSI      23
#define SPI_MISO      19
#define SPI_SCK       18
// DAC MAX9835
#define I2S_DOUT      25
#define I2S_BCLK      27
#define I2S_LRC       26

Audio audio;
```

En el setup inicialitzem el pin SD_CS com a sortida i a nivell alt. Inicialitzem també el bus SPI amb els seus pins (MOSI, MISO i SCK). Posteriorment, posem la velocitat de comunicació usual i iniciem la targeta SD amn el seu pin SD_CS. Després, dins de l'objecte audio, definim els pins de sortida (BCLK, LRC, DOUT), posem el volum a 10 i realitzem la connexió per a escollir l'arxiu de la SD, en el nostre cas anomenat "testaudio.wav".
```c
void setup(){
    pinMode(SD_CS, OUTPUT);
    digitalWrite(SD_CS, HIGH);
    SPI.begin(SPI_SCK, SPI_MISO, SPI_MOSI);
    Serial.begin(115200);
    SD.begin(SD_CS);
    audio.setPinout(I2S_BCLK, I2S_LRC, I2S_DOUT);
    audio.setVolume(10);
    audio.connecttoFS(SD, "testaudio.wav");
}
```

Per a acabar, en el loop es crida a la funció audio, que s'anirà repetint en bucle.
```c
void loop(){
    audio.loop();
}
```

### Part B: Codi complet
```c
#include "Audio.h"
#include "SD.h"
#include "FS.h"

// Digital I/O used, Tarjeta SD
#define SD_CS          5
#define SPI_MOSI      23
#define SPI_MISO      19
#define SPI_SCK       18
// DAC MAX9835
#define I2S_DOUT      25
#define I2S_BCLK      27
#define I2S_LRC       26

Audio audio;

void setup(){
    pinMode(SD_CS, OUTPUT);
    digitalWrite(SD_CS, HIGH);
    SPI.begin(SPI_SCK, SPI_MISO, SPI_MOSI);
    Serial.begin(115200);
    SD.begin(SD_CS);
    audio.setPinout(I2S_BCLK, I2S_LRC, I2S_DOUT);
    audio.setVolume(10); // 0...21
    audio.connecttoFS(SD, "testaudio.wav");
}

void loop(){
    audio.loop();
}
```

## Part EXTRA: Reproducció d'una senyal de ràdio per internet
El codi de l'apartat extra s'ha extret del exemple de WebRadio de la ESP8266Audio (.pio\libdeps/esp32dev\ESP8266Audio\examples\WebRadio), degut a això aquest ja està correctament comentat i es pot veure el seu funcionament si es llegeix.

### Part EXTRA: Codi complet
```c
#include <Arduino.h>
#if defined(ARDUINO_ARCH_RP2040)
// Nothing here
#else
#ifdef ESP32
    #include <WiFi.h>
#else
    #include <ESP8266WiFi.h>
#endif
#include "web.h"

void WebPrintError(WiFiClient *client, int code)
{
  switch(code) {
    case 301: WebPrintf(client, "301 Moved Permanently"); break;
    case 400: WebPrintf(client, "400 Bad Request"); break;
    case 401: WebPrintf(client, "401 Unauthorized"); break;
    case 404: WebPrintf(client, "404 Not Found"); break;
    case 405: WebPrintf(client, "405 Method Not Allowed"); break;
    default:  WebPrintf(client, "500 Server Error"); break;
  }
}

void WebError(WiFiClient *client, int code, const char *headers, bool usePMEM)
{
  WebPrintf(client, "HTTP/1.1 %d\r\n", code);
  WebPrintf(client, "Server: PsychoPlug\r\n");
  WebPrintf(client, "Content-type: text/html\r\n");
  WebPrintf(client, "Cache-Control: no-cache, no-store, must-revalidate\r\n");
  WebPrintf(client, "Pragma: no-cache\r\n");
  WebPrintf(client, "Expires: 0\r\n");
  WebPrintf(client, "Connection: close\r\n");
  if (headers) {
    if (!usePMEM) {
      WebPrintf(client, "%s", headers);
    } else {
      WebPrintfPSTR(client, headers);
    }
  }
  WebPrintf(client, "\r\n\r\n");
  WebPrintf(client, DOCTYPE);
  WebPrintf(client, "<html><head><title>");
  WebPrintError(client, code);
  WebPrintf(client, "</title>" ENCODING "</head>\n");
  WebPrintf(client, "<body><h1>");
  WebPrintError(client, code);
  WebPrintf(client, "</h1></body></html>\r\n");
}

void WebHeaders(WiFiClient *client, PGM_P /*const char **/headers)
{
  WebPrintf(client, "HTTP/1.1 200 OK\r\n");
  WebPrintf(client, "Server: PsychoPlug\r\n");
  WebPrintf(client, "Content-type: text/html\r\n");
  WebPrintf(client, "Cache-Control: no-cache, no-store, must-revalidate\r\n");
  WebPrintf(client, "Pragma: no-cache\r\n");
  WebPrintf(client, "Connection: close\r\n");
  WebPrintf(client, "Expires: 0\r\n");
  if (headers) {
    WebPrintfPSTR(client, headers);
  }
  WebPrintf(client, "\r\n");
}

// In-place decoder, overwrites source with decoded values.  Needs 0-termination on input
// Try and keep memory needs low, speed not critical
static uint8_t b64lut(uint8_t i)
{
  if (i >= 'A' && i <= 'Z') return i - 'A';
  if (i >= 'a' && i <= 'z') return i - 'a' + 26;
  if (i >= '0' && i <= '9') return i - '0' + 52;
  if (i == '-') return 62;
  if (i == '_') return 63;
  else return 64;// sentinel
}

void Base64Decode(char *str)
{
  char *dest;
  dest = str;

  if (strlen(str)%4) return; // Not multiple of 4 == error
  
  while (*str) {
    uint8_t a = b64lut(*(str++));
    uint8_t b = b64lut(*(str++));
    uint8_t c = b64lut(*(str++));
    uint8_t d = b64lut(*(str++));
    *(dest++) = (a << 2) | ((b & 0x30) >> 4);
    if (c == 64) break;
    *(dest++) = ((b & 0x0f) << 4) | ((c & 0x3c) >> 2);
    if (d == 64) break;
    *(dest++) = ((c & 0x03) << 6) | d;
  }
  *dest = 0; // Terminate the string
}

void URLDecode(char *ptr)
{
  while (*ptr) {
    if (*ptr == '+') {
      *ptr = ' ';
    } else if (*ptr == '%') {
      if (*(ptr+1) && *(ptr+2)) {
        byte a = *(ptr + 1);
        byte b = *(ptr + 2);
        if (a>='0' && a<='9') a -= '0';
        else if (a>='a' && a<='f') a = a - 'a' + 10;
        else if (a>='A' && a<='F') a = a - 'A' + 10;
        if (b>='0' && b<='9') b -= '0';
        else if (b>='a' && b<='f') b = b - 'a' + 10;
        else if (b>='A' && b<='F') b = b - 'A' + 10;
        *ptr = ((a&0x0f)<<4) | (b&0x0f);
        // Safe strcpy the rest of the string back
        char *p1 = ptr + 1;
        char *p2 = ptr + 3;
        while (*p2) { *p1 = *p2; p1++; p2++; }
        *p1 = 0;
      }
      // OTW this is a bad encoding, just pass unchanged
    }
    ptr++;
  }
}

// Parse HTTP request
bool WebReadRequest(WiFiClient *client, char *reqBuff, int reqBuffLen, char **urlStr, char **paramStr)
{
  static char NUL = 0; // Get around writable strings...

  *urlStr = NULL;
  *paramStr = NULL;

  unsigned long timeoutMS = millis() + 5000; // Max delay before we timeout
  while (!client->available() && millis() < timeoutMS) { delay(10); }
  if (!client->available()) {
    return false;
  }
  int wlen = client->readBytesUntil('\r', reqBuff, reqBuffLen-1);
  reqBuff[wlen] = 0;

  
  // Delete HTTP version (well, anything after the 2nd space)
  char *ptr = reqBuff;
  while (*ptr && *ptr!=' ') ptr++;
  if (*ptr) ptr++;
  while (*ptr && *ptr!=' ') ptr++;
  *ptr = 0;

  URLDecode(reqBuff);

  char *url;
  char *qp;
  if (!memcmp_P(reqBuff, PSTR("GET "), 4)) {
    client->flush(); // Don't need anything here...
    
    // Break into URL and form data
    url = reqBuff+4;
    while (*url && *url=='/') url++; // Strip off leading /s
    qp = strchr(url, '?');
    if (qp) {
      *qp = 0; // End URL
      qp++;
    } else {
      qp = &NUL;
    }
  } else if (!memcmp_P(reqBuff, PSTR("POST "), 5)) {
    uint8_t newline;
    client->read(&newline, 1); // Get rid of \n

    url = reqBuff+5;
    while (*url && *url=='/') url++; // Strip off leading /s
    qp = strchr(url, '?');
    if (qp) *qp = 0; // End URL @ ?
    // In a POST the params are in the body
    int sizeleft = reqBuffLen - strlen(reqBuff) - 1;
    qp = reqBuff + strlen(reqBuff) + 1;
    int wlen = client->readBytesUntil('\r', qp, sizeleft-1);
    qp[wlen] = 0;
    client->flush();
    URLDecode(qp);
  } else {
    // Not a GET or POST, error
    WebError(client, 405, PSTR("Allow: GET, POST"));
    return false;
  }

  if (urlStr) *urlStr = url;
  if (paramStr) *paramStr = qp;

  return true;
}

// Scan out and update a pointeinto the param string, returning the name and value or false if done
bool ParseParam(char **paramStr, char **name, char **value)
{
  char *data = *paramStr;
 
  if (*data==0) return false;
  
  char *namePtr = data;
  while ((*data != 0) && (*data != '=') && (*data != '&')) data++;
  if (*data) { *data = 0; data++; }
  char *valPtr = data;
  if  (*data == '=') data++;
  while ((*data != 0) && (*data != '=') && (*data != '&')) data++;
  if (*data) { *data = 0; data++;}
  
  *paramStr = data;
  *name = namePtr;
  *value = valPtr;

  return true;
}

bool IsIndexHTML(const char *url)
{
  if (!url) return false;
  if (*url==0 || !strcmp_P(url, PSTR("/")) || !strcmp_P(url, PSTR("/index.html")) || !strcmp_P(url, PSTR("index.html"))) return true;
  else return false;
}

void WebFormText(WiFiClient *client, /*const char **/ PGM_P label, const char *name, const char *value, bool enabled)
{
  WebPrintfPSTR(client, label);
  WebPrintf(client, ": <input type=\"text\" name=\"%s\" id=\"%s\" value=\"%s\" %s><br>\n", name, name, value, !enabled?"disabled":"");
}
void WebFormText(WiFiClient *client, /*const char **/ PGM_P label, const char *name, const int value, bool enabled)
{
  WebPrintfPSTR(client, label);
  WebPrintf(client, ": <input type=\"text\" name=\"%s\" id=\"%s\" value=\"%d\" %s><br>\n", name, name, value, !enabled?"disabled":"");
}
void WebFormCheckbox(WiFiClient *client, /*const char **/ PGM_P label, const char *name, bool checked, bool enabled)
{
  WebPrintf(client, "<input type=\"checkbox\" name=\"%s\" id=\"%s\" %s %s> ", name, name, checked?"checked":"", !enabled?"disabled":"");
  WebPrintfPSTR(client, label);
  WebPrintf(client, "<br>\n");
}
void WebFormCheckboxDisabler(WiFiClient *client, PGM_P /*const char **/label, const char *name, bool invert, bool checked, bool enabled, const char *ids[])
{
  WebPrintf(client,"<input type=\"checkbox\" name=\"%s\" id=\"%s\" onclick=\"", name,name);
  if (invert) WebPrintf(client, "var x = true; if (this.checked) { x = false; }\n")
  else WebPrintf(client, "var x = false; if (this.checked) { x = true; }\n");
  for (byte i=0; ids[i][0]; i++ ) {
    WebPrintf(client, "document.getElementById('%s').disabled = x;\n", ids[i]);
  }
  WebPrintf(client, "\" %s %s> ", checked?"checked":"", !enabled?"disabled":"")
  WebPrintfPSTR(client, label);
  WebPrintf(client, "<br>\n");
}

// Scan an integer from a string, place it into dest, and then return # of bytes scanned
int ParseInt(char *src, int *dest)
{
  byte count = 0;
  bool neg = false;
  int res = 0;
  if (!src) return 0;
  if (src[0] == '-') {neg = true; src++; count++;}
  while (*src && (*src>='0') && (*src<='9')) {
    res = res * 10;
    res += *src - '0';
    src++;
    count++;
  }
  if (neg) res *= -1;
  if (dest) *dest = res;
  return count;
}

void Read4Int(char *str, byte *p)
{
  int i;
  str += ParseInt(str, &i); p[0] = i; if (*str) str++;
  str += ParseInt(str, &i); p[1] = i; if (*str) str++;
  str += ParseInt(str, &i); p[2] = i; if (*str) str++;
  str += ParseInt(str, &i); p[3] = i;
}
#endif
```
