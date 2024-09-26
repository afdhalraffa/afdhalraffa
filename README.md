//MOSI: Pin 23 / ICSP-4 Abu-abu
//MISO: Pin 19 / ICSP-1 Ungu
//SCK: Pin 18 / ISCP-3 Putih
//SS/SDA: Pin 5 Hitam
//RST: Pin 27 Hijau
//3.3V Kuning
//GND Biru

/*
   -- New project --
   
   This source code of graphical user interface 
   has been generated automatically by RemoteXY editor.
   To compile this code using RemoteXY library 3.1.13 or later version 
   download by link http://remotexy.com/en/library/
   To connect using RemoteXY mobile app by link http://remotexy.com/en/download/                   
     - for ANDROID 4.14.08 or later version;
     - for iOS 1.11.2 or later version;
    
   This source code is free software; you can redistribute it and/or
   modify it under the terms of the GNU Lesser General Public
   License as published by the Free Software Foundation; either
   version 2.1 of the License, or (at your option) any later version.    
*/

//////////////////////////////////////////////
//        RemoteXY include library          //
//////////////////////////////////////////////

// you can enable debug logging to Serial at 115200
//#define REMOTEXY__DEBUGLOG    

// RemoteXY select connection mode and include library 
#define REMOTEXY_MODE__ESP32CORE_BLE

#include <BLEDevice.h>

// RemoteXY connection settings 
#define REMOTEXY_BLUETOOTH_NAME "KuDAsaI"


#include <RemoteXY.h>

// RemoteXY GUI configuration  
#pragma pack(push, 1)  
uint8_t RemoteXY_CONF[] =   // 46 bytes
  { 255,0,0,93,0,39,0,18,0,0,0,31,1,106,200,1,1,3,0,67,
  7,26,90,10,68,2,26,31,67,7,45,90,10,68,2,26,31,67,7,64,
  90,10,68,2,26,31 };
  
// this structure defines all the variables and events of your control interface 
struct {

    // output variables
  char Kursi_D[31]; // string UTF8 end zero
  char Kursi_L[31]; // string UTF8 end zero
  char Kursi_I[31]; // string UTF8 end zero

    // other variable
  uint8_t connect_flag;  // =1 if wire connected, else =0

} RemoteXY;   
#pragma pack(pop)
 
/////////////////////////////////////////////
//           END RemoteXY include          //
/////////////////////////////////////////////


#include <RFID.h>
#include <SPI.h>
#include <ESP32Servo.h>  // Pustaka untuk kontrol servo

#define SS_PIN 5    // Pin RFID
#define RST_PIN 27  // Pin RFID
RFID rfid(SS_PIN, RST_PIN);

// Pin motor servo
#define SERVO1_PIN 33
#define SERVO2_PIN 32
#define SERVO3_PIN 26  // Tambahkan pin untuk servo 3

// Inisialisasi objek servo
Servo kursi_lansia;
Servo kursi_ibu_hamil;
Servo kursi_disabilitas;

// Definisi kartu Lansia & Disabilitas
int cards[3][5] = {
  { 115, 15, 63, 52, 119 },  // Kartu Lansia & Disabilitas 1
  { 122, 232, 21, 128, 7 },  // Kartu Lansia & Disabilitas 2
  { 5, 152, 197, 3, 91 },    // Kartu Lansia & Disabilitas 3
};

// Variabel untuk melacak status servo dan kartu
bool servo1Active = false;
bool servo2Active = false;
bool servo3Active = false;
bool card1Active = false;  // Status kartu 1
bool card2Active = false;  // Status kartu 2
bool card3Active = false;  // Status kartu 3
bool kursi = false;

void setup() {
  RemoteXY_Init (); 
  Serial.begin(9600);
  SPI.begin();
  rfid.init();

  // Inisialisasi servo
  kursi_lansia.attach(SERVO1_PIN);
  kursi_ibu_hamil.attach(SERVO2_PIN);
  kursi_disabilitas.attach(SERVO3_PIN);

  // Posisi awal servo
  kursi_lansia.write(90);       // Posisi awal servo 1
  kursi_ibu_hamil.write(90);    // Posisi awal servo 2
  kursi_disabilitas.write(90);  // Posisi awal servo 3
}

void loop() {
  RemoteXY_Handler ();
  if (rfid.isCard()) {
    if (rfid.readCardSerial()) {
      // Membaca nomor kartu RFID
      Serial.print(rfid.serNum[0]);
      Serial.print(" ");
      Serial.print(rfid.serNum[1]);
      Serial.print(" ");
      Serial.print(rfid.serNum[2]);
      Serial.print(" ");
      Serial.print(rfid.serNum[3]);
      Serial.print(" ");
      Serial.print(rfid.serNum[4]);
      Serial.println(" ");

      bool diterima = false;
      int cardIndex = -1;  // Variabel untuk menyimpan indeks kartu yang terdeteksi

      // Memeriksa apakah kartu yang dibaca ada dalam daftar kartu yang diterima
      for (int x = 0; x < sizeof(cards) / sizeof(cards[0]); x++) {
        bool match = true;
        for (int i = 0; i < sizeof(rfid.serNum) / sizeof(rfid.serNum[0]); i++) {
          if (rfid.serNum[i] != cards[x][i]) {
            match = false;
            break;
          }
        }
        if (match) {
          diterima = true;
          cardIndex = x;  // Simpan indeks kartu yang terdeteksi
          break;
        }
      }

      if (kursi_lansia.read()==-1){
        kursi = true;
      }
      Serial.println(kursi_lansia.read());
      if (diterima) {
        Serial.println("Lansia, Ibu Hamil & Disabilitas");

        // Aktifkan atau nonaktifkan motor servo berdasarkan kartu yang terdeteksi
        if (cardIndex == 0) {  // Kartu 1
          if (!card1Active) {
            kursi_lansia.write(0);  // Aktifkan servo 1
            Serial.println("KURSI LANSIA AKTIF");
            strcpy(RemoteXY.Kursi_L, "KURSI LANSIA AKTIF");
            servo1Active = true;
            card1Active = true;
          } else {
            kursi_lansia.write(70);  // Nonaktifkan servo 1
            Serial.println("KURSI LANSIA NONAKTIF");
            strcpy(RemoteXY.Kursi_L, "KURSI LANSIA TIDAK AKTIF");
            servo1Active = false;
            card1Active = false;
          }
        } else if (cardIndex == 1) {  // Kartu 2
          if (!card2Active) {
            kursi_ibu_hamil.write(0);  // Aktifkan servo 2
            Serial.println("KURSI IBU HAMIL AKTIF");
            strcpy(RemoteXY.Kursi_I, "KURSI IBU HAMIL AKTIF");
            servo2Active = true;
            card2Active = true;
          } else {
            kursi_ibu_hamil.write(70);  // Nonaktifkan servo 2
            Serial.println("KURSI IBU HAMIL NONAKTIF");
            strcpy(RemoteXY.Kursi_I, "KURSI IBU HAMIL NONAKTIF");
            servo2Active = false;
            card2Active = false;
          }
        } else if (cardIndex == 2) {  // Kartu 3
          if (!card3Active) {
            kursi_disabilitas.write(0);  // Aktifkan servo 3
            Serial.println("KURSI DISABILITAS AKTIF");
            strcpy(RemoteXY.Kursi_D, "KURSI DISABILITAS AKTIF");
            servo3Active = true;
            card3Active = true;
          } else {
            kursi_disabilitas.write(70);  // Nonaktifkan servo 3
            Serial.println("KURSI DISABILITAS NONAKTIF");
            strcpy(RemoteXY.Kursi_D, "KURSI DISABILITAS NONAKTIF");
            servo3Active = false;
            card3Active = false;
          }
        }
      } else {
        Serial.println("Bukan Lansia, Ibu Hamil & Disabilitas");
      }

      rfid.halt();  // Menghentikan pembacaan kartu
    }
  } else {
    // Jika tidak ada kartu yang terdeteksi, tampilkan pesan
    Serial.println("Silahkan Tap Kartu");
  }

  delay(500);  // Delay untuk mencegah pembacaan yang terlalu cepat
}
