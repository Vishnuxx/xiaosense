/*
  Make sure to turn on psram to run this program
*/

#include <SD.h>
#include <I2S.h>


#define SAMPLE_RATE 16000U
#define SAMPLE_BITS 16
#define WAV_HEADER_SIZE 44
#define VOLUME_GAIN 2

#define PWM_PIN 5 //change the pin as you like

//adjust cheyyanam
int PWM_FREQUENCY = 10000;
int PWM_DUTY_CYCLE = 128;

void generate_wav_header(uint8_t *wav_header, uint32_t wav_size, uint32_t sample_rate) {
  uint32_t file_size = wav_size + WAV_HEADER_SIZE - 8;
  uint32_t byte_rate = sample_rate * SAMPLE_BITS / 8;
  const uint8_t set_wav_header[] = {
    'R', 'I', 'F', 'F',                                                                                                                                           // ChunkID
    static_cast<uint8_t>(file_size), static_cast<uint8_t>(file_size >> 8), static_cast<uint8_t>(file_size >> 16), static_cast<uint8_t>(file_size >> 24),          // ChunkSize
    'W', 'A', 'V', 'E',                                                                                                                                           // Format
    'f', 'm', 't', ' ',                                                                                                                                           // Subchunk1ID
    0x10, 0x00, 0x00, 0x00,                                                                                                                                       // Subchunk1Size (16 for PCM)
    0x01, 0x00,                                                                                                                                                   // AudioFormat (1 for PCM)
    0x01, 0x00,                                                                                                                                                   // NumChannels (1 channel)
    static_cast<uint8_t>(sample_rate), static_cast<uint8_t>(sample_rate >> 8), static_cast<uint8_t>(sample_rate >> 16), static_cast<uint8_t>(sample_rate >> 24),  // SampleRate
    static_cast<uint8_t>(byte_rate), static_cast<uint8_t>(byte_rate >> 8), static_cast<uint8_t>(byte_rate >> 16), static_cast<uint8_t>(byte_rate >> 24),          // ByteRate
    0x02, 0x00,                                                                                                                                                   // BlockAlign
    0x10, 0x00,                                                                                                                                                   // BitsPerSample (16 bits)
    'd', 'a', 't', 'a',                                                                                                                                           // Subchunk2ID
    static_cast<uint8_t>(wav_size), static_cast<uint8_t>(wav_size >> 8), static_cast<uint8_t>(wav_size >> 16), static_cast<uint8_t>(wav_size >> 24),              // Subchunk2Size
  };
  memcpy(wav_header, set_wav_header, sizeof(set_wav_header));
}




//filename : filename with .wav , duration: non negative numbers max 240

void recordAudio(String filename, size_t duration) {
  size_t record_size = ((SAMPLE_RATE * SAMPLE_BITS) / 8) * duration;
  uint8_t *rec_buffer = (uint8_t *)ps_malloc(record_size);
  if (rec_buffer == NULL) {
    Serial.println("malloc failed");
    return;
  }

  size_t sample_size = 0;

  Serial.println("Recording started...");
  esp_i2s::i2s_read(esp_i2s::I2S_NUM_0, rec_buffer, record_size, &sample_size, portMAX_DELAY);

  if (sample_size != 0) {
    Serial.printf("Record %d bytes\n", sample_size);
  } else {
    free(rec_buffer);
    Serial.println("Record Failed!");
    return;
  }
  File file = SD.open(filename, FILE_WRITE);
  uint8_t wav_header[WAV_HEADER_SIZE];
  generate_wav_header(wav_header, record_size, SAMPLE_RATE);
  Serial.println("Generated wav header");
  for (uint32_t i = 0; i < record_size; i += SAMPLE_BITS / 8) {
    (*(uint16_t *)(rec_buffer + i)) <<= VOLUME_GAIN;
  }
  Serial.println("Increased volume");
  file.write(wav_header, WAV_HEADER_SIZE);
  size_t size = file.write(rec_buffer, record_size);
  if (size != record_size) {
    Serial.printf("Write file Failed! size: %d , recsize: %d\n", size, record_size);
  } else {
    Serial.println("Audio Written successfully to sd card");
  }
  file.close();
  free(rec_buffer);
}



void play_wav(String filename, uint8_t wav_header_size) {

  File audioFile = SD.open(filename);
  if (!audioFile) {
    Serial.println("Failed to open audio file");
    return;
  }

  audioFile.seek(wav_header_size);


  while (audioFile.available()) {

    uint8_t sample = audioFile.read();

    int dutyCycle = map(sample, 0, 255, 0, 255);

    ledcWrite(0, dutyCycle);
  }

  // Close the audio file
  audioFile.close();
}



void
setup() {
  Serial.begin(115200);
  if (!SD.begin(21)) {
    Serial.println("failed to initialize SD");
    while (1)
      ;
  }
  I2S.setAllPins(-1, 42, 41, -1, -1);
  if (!I2S.begin(PDM_MONO_MODE, SAMPLE_RATE, SAMPLE_BITS)) {
    Serial.println("Failed to initialize I2S!");
    while (1)
      ;
  }

  // Set PWM pin as an output
  pinMode(PWM_PIN, OUTPUT);
  // Initialize PWM
  ledcSetup(0, PWM_FREQUENCY, 8);  // channel 0, 10 kHz, 8-bit resolution
  ledcAttachPin(PWM_PIN, 0);       // attach channel 0 to pin

  recordAudio("/file1.wav", 15);
}

void loop() {
  // put your main code here, to run repeatedly:
}
