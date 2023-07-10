#include <OneWire.h> //Mengimport library OneWire.h yang digunakan untuk komunikasi dengan sensor yang menggunakan protokol OneWire.
#include <DallasTemperature.h>  //Mengimport library DallasTemperatur.h yang digunakan untuk membaca suhu dari sensor Dallas.
#include <BlynkSimpleEsp8266.h> //Mengimport library BlynkSimpleEsp8266.h yang digunakan untuk menghubungkan perangkat ke platform Blynk.


#define sensor_ds18b20 D4 //Mendefinisikan sensor DS18B20 sebagai D4
OneWire oneWire(sensor_ds18b20);  //Membuat objek OneWire menggunakan pin D4
DallasTemperature sensors(&oneWire);  //Membuat objek sensor dari kelas Dallas Temperatur dengan menggunakan objek OneWire sebelumnya

const int ph_Pin = A0; //Mendefinisikan pin A0 sebagai pin analog untuk membaca data dari sensor pH
float Po = 0; //Mendefinisikan variabel Po (pH offset) dengan nilai 0
float PH_step;  // membuat variabel yang akan menyimpan perubahan pH
int nilai_analog_PH;  // mendefinisikan variabel untuk menyimpan nilai analog pembacaan sensor pH
double TeganganPh;  //mendefinisikan variabel untuk menyimpan tegangan pembacaan sensor pH

// untuk kalibrasi
float PH4 = 3.1;
float PH7 = 4.23;

char auth[] = "CQvKkfHxIH5q-hGGSn8Z3DPi04FmwCzf"; //mendefinisikan token otorisasi yang menghubungkan perangkat dengan platform blynk
char ssid[] = "LEYS"; //mendefinisikan nama dari wifi yang digunakan
char pass[] = "Ba1ley99"; //mendefinisikan password dari wifi yang digunakan

void setup() {
  pinMode(ph_Pin, INPUT); //mengatur pin ph_pin menjadi input yang membaca data dari sensor pH
  Serial.begin(9600); // memulai komunikasi serial dengan kecepatan baud rate 9600 untuk debugging dan output serial
  sensors.begin();  //memulai komunikasi dengan sensor suhu dallas

  Blynk.begin(auth, ssid, pass);  //menghubungkan perangkat dengan platform blynk yang sudah didefinisikan sebelumnya
}

void loop() {
  Blynk.run();  // memproses semua perintah blynk dan mempertahankan koneksi dengan platform blynk

  sensors.requestTemperatures();  //mengirim perintah ke sensor suhu dallas untuk memperbaharui data
  static unsigned long timepoint = millis();  //mendeklarasikan variable timepoint sebagai unsigned long dan menginisialisasinya dengan nilai millis saat ini
  if (millis() - timepoint > 1000U) { //memeriksa selisih waktu millis dengan waktu terakhir
    timepoint = millis(); // memperbaharui nilai timepoint dengan waktu saat ini
    float tegangan = analogRead(ph_Pin) / 1024.0 * 5000;  //membaca nilai tegangan dari sensor ph menggunakan fungsi analogRead() dan mengkonversinya menjadi nilai tegangan yang sebenarnya
    float suhu = sensors.getTempCByIndex(0);  //membaca suhu dari sensor suhu dengan fungsi getTempCByIndex()

    Serial.print("Celsius temperature: ");  //mencetak teks ke serial
    Serial.print(suhu, 1);  // mencetak nilai suhu ke serial
    Serial.println("°C"); //mencetak teks ke serial

    int nilai_analog_PH = analogRead(ph_Pin); //membaca nilai analog dari sensor ph dan menyimpan ke variabel nilai_analog_PH
    Serial.print("Nilai ADC Ph : ");  //mencetak teks ke serial
    Serial.println(nilai_analog_PH);  //mencetak nilai dan membuat baris baru pada serial
    double TeganganPh = (3.3 / 1023.0) * nilai_analog_PH; //menghitung tegangan dari nilai analog ph menggunakan rumus dan nilai disimpan ke variable TeganganPH
    Serial.print("Tegangan PH : "); //mencetak teks ke serial
    Serial.println(TeganganPh, 3);  // mencetak nilai ke serial

    PH_step = (PH7 - PH4) / 3;  //menghitung langkah perubahan ph dengan rumus dan nilainya disimpan ke variable PH_step
    Po = 7.0 - ((TeganganPh - PH4) / PH_step); // Memperbaiki rumus perhitungan Po
    Serial.print("Nilai PH cairan : "); //mencetak teks ke serial
    Serial.println(Po, 2);  //mencetak nilai ke serial

    Blynk.virtualWrite(V1, String("Suhu: ") + suhu + "°C"); //mengirim data suhu ke widget virtual pin v1 di platform blynk
    Blynk.virtualWrite(V2, String("pH: ") + Po);  //mengirim data ph ke widget virtual pin v2 di platform blynk
    Blynk.virtualWrite(V3, String("Keterangan: ") + getpHLevelDescription(Po)); //mengirim keterangan level ph ke widget virtual pin v3 di platform blynk dengan memanggil fungi getpHLevelDescription() untuk mendapatkan keterangan ph 
  }
  delay(100); //memberikan delay sebelum mengeksekusi iterasi berikutnya dari program yang berada di loop
}

String getpHLevelDescription(float pH) {  //mendefinisikan fungsi getpHLevelDescription yang menerima nilai ph sebagai argumen dan mengembalikan tipe data string
  // melakukan pengecekan kondisi dari ph
  if (pH == 7) {
    return "Netral";
  } else if (pH < 7 && pH >= 0) {
    return "Asam";
  } else if (pH > 7 && pH <= 14) {
    return "Basa";
  } else {
    return "Sensor tidak terbaca";
  }
}
