```C++
#include <PN532.h>
#include <SPI.h>

// 定义芯片选择引脚，可连接至D10或D9
#define PN532_CS 10

// 初始化PN532对象
PN532 nfc(PN532_CS);

// 调试模式开关
#define NFC_DEMO_DEBUG 1

void setup() {
#ifdef NFC_DEMO_DEBUG
  Serial.begin(9600);
  Serial.println("Hello!");
#endif

  nfc.begin();

  // 获取固件版本信息
  uint32_t versiondata = nfc.getFirmwareVersion();
  if (!versiondata) {
#ifdef NFC_DEMO_DEBUG
    Serial.println("Didn't find PN53x board");
#endif
    while (1); // 停止程序
  }

#ifdef NFC_DEMO_DEBUG
  // 打印芯片和固件版本信息
  Serial.print("Found chip PN5");
  Serial.println((versiondata >> 24) & 0xFF, HEX);
  Serial.print("Firmware ver. ");
  Serial.print((versiondata >> 16) & 0xFF, DEC);
  Serial.print('.');
  Serial.println((versiondata >> 8) & 0xFF, DEC);
  Serial.print("Supports ");
  Serial.println(versiondata & 0xFF, HEX);
#endif

  // 配置RFID读取模块
  nfc.SAMConfig();
}

void loop() {
  // 查找MIFARE类型卡片
  uint32_t id = nfc.readPassiveTargetID(PN532_MIFARE_ISO14443A);

  if (id != 0) {
#ifdef NFC_DEMO_DEBUG
    Serial.print("Read card #");
    Serial.println(id);
#endif

    // 使用默认密钥进行验证
    uint8_t keys[] = { 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF };
    bool authenticated = nfc.authenticateBlock(1, id, 0x08, KEY_A, keys);

    if (authenticated) {
      uint8_t block[16];  // 存储读取的块数据

      // 读取内存块0x08的数据
      if (nfc.readMemoryBlock(1, 0x08, block)) {
#ifdef NFC_DEMO_DEBUG
        // 打印读取的块数据
        for (uint8_t i = 0; i < 16; i++) {
          Serial.print(block[i], HEX);
          Serial.print(" ");
        }
        Serial.println();
#endif
      }
    }
  }
  delay(500); // 延迟500毫秒
}
```
