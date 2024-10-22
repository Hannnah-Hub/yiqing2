```C++
#include <Adafruit_PN532.h>
#include <SPI.h>

// 定义 Key A 和 Key B
#define KEY_A 0x60  
#define KEY_B 0x61  

#define PN532_CS 10
Adafruit_PN532 nfc(PN532_CS);

#define NFC_DEMO_DEBUG 1

void setup() {
#ifdef NFC_DEMO_DEBUG
    Serial.begin(9600);
    Serial.println("Hello!");
#endif

    nfc.begin();
    uint32_t versiondata = nfc.getFirmwareVersion();
    if (!versiondata) {
#ifdef NFC_DEMO_DEBUG
        Serial.println("Didn't find PN53x board");
#endif
        while (1);
    }

#ifdef NFC_DEMO_DEBUG
    Serial.print("Found chip PN5");
    Serial.println((versiondata >> 24) & 0xFF, HEX);
    Serial.print("Firmware ver. ");
    Serial.print((versiondata >> 16) & 0xFF, DEC);
    Serial.print('.');
    Serial.println((versiondata >> 8) & 0xFF, DEC);
    Serial.print("Supports ");
    Serial.println(versiondata & 0xFF, HEX);
#endif

    nfc.SAMConfig();
}

void loop() {
    uint8_t uid[7];  
    uint8_t uidLength;
    if (nfc.readPassiveTargetID(PN532_MIFARE_ISO14443A, uid, &uidLength)) {
#ifdef NFC_DEMO_DEBUG
        Serial.print("Read card with UID: ");
        for (uint8_t i = 0; i < uidLength; i++) {
            Serial.print(uid[i], HEX);
            Serial.print(" ");
        }
        Serial.println();
#endif

        uint8_t keys[] = { 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF };
        bool authenticated = nfc.mifareclassic_AuthenticateBlock(uid, uidLength, 4, KEY_A, keys);

        if (authenticated) {
            uint8_t block[16];
            if (nfc.mifareclassic_ReadDataBlock(4, block)) {
                Serial.print("Data in Block 4: ");
                for (uint8_t i = 0; i < 16; i++) {
                    Serial.print(block[i], HEX);
                    Serial.print(" ");
                }
                Serial.println();
            } else {
                Serial.println("Failed to read block 4");
            }
        } else {
            Serial.println("Authentication failed");
        }
    }
    delay(500);
}
```
