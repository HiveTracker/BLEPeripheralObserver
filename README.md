# Arduino BLEPeripheral

An [Arduino](http://arduino.cc) library for creating custom BLE peripherals.

## Compatible Hardware

### [Nordic Semiconductor nRF8001](http://www.nordicsemi.com/eng/Products/Bluetooth-R-low-energy/nRF8001)

 * [Adafruit](http://www.adafruit.com) 
   * [Bluefruit LE - nRF8001 Breakout](http://www.adafruit.com/products/1697)
 * [RedBearLab](http://redbearlab.com)
   * [BLE Shield](http://redbearlab.com/bleshield/)
   * [Blend Micro](http://redbearlab.com/blendmicro/)
   * [Blend](http://redbearlab.com/blend/)

**Note:** Does not require use of [nRFgo Studio](http://www.nordicsemi.com/chi/node_176/2.4GHz-RF/nRFgo-Studio)!

#### Pinouts

| Shield/Board | REQ Pin | RDY Pin | RST Pin |
| Bluefruit LE | 10 | 2 | 9 |
| BLE Shield | 9 | 8 | UNUSED |
| Blend Micro | 6 | 7 | UNUSED |

## Usage

### Download Library
```
cd ~/Documents/Arduino/libraries/
git clone https://github.com/sandeepmistry/arduino-BLEPeripheral BLEPeripheral
```

### Starter sketch
```
// Import libraries (BLEPeripheral depends on SPI)
#include <SPI.h>
#include <BLEPeripheral.h>

// define pins (varies per shield/board)
#define BLE_REQ   10
#define BLE_RDY   2
#define BLE_RST   9

// create peripheral instance, see pinouts above
BLEPeripheral blePeripheral = BLEPeripheral(BLE_REQ, BLE_RDY, BLE_RST);

// uuid's can be:
//   16-bit: "ffff"
//  128-bit: "19b10010e8f2537e4f6cd104768a1214" (dashed format also supported)

// create one or more services
BLEService service = BLEService("fff0");

// create one or more characteristics
BLECharacteristicT<short> characteristic = BLECharacteristicT<short>("fff1", BLEPropertyRead | BLEPropertyWrite);

// create one or more descriptors (optional)
BLEDescriptor descriptor = BLEDescriptor("2901", "value");

void setup() {
  Serial.begin(115200);
#if defined (__AVR_ATmega32U4__)
  //Wait until the serial port is available (useful only for the Leonardo)
  //As the Leonardo board is not reseted every time you open the Serial Monitor
  while(!Serial) {}
  delay(5000);  //5 seconds delay for enabling to see the start up comments on the serial board
#endif

  blePeripheral.setLocalName("local-name"); // optional
  blePeripheral.setAdvertisedServiceUuid(service.uuid()); // optional

  // add attributes (services, characteristics, descriptors) to peripheral
  blePeripheral.addAttribute(service);
  blePeripheral.addAttribute(characteristic);
  blePeripheral.addAttribute(descriptor);

  // set initial value
  characteristic.setValue(0);

  // begin initialization
  blePeripheral.begin();
}

void loop() {
  // poll for events
  blePeripheral.poll();

  if (blePeripheral.isConnected()) {
    // central connected to peripheral
    if (characteristic.hasNewValue()) {
      // central wrote new value to characteristic

      Serial.println(characteristic.value());
    }

    // set value on characteristic
    characteristic.setValue(5);
  }
}


```

## API
See [API.md](API.md).

## Examples
See [examples](examples) folder.

## Memory usage

Creating dynamic setup messages for the nRF8001 uses a quite a lot of memory.

 * base services/characteristics: ~229 bytes
 * per characteristic: up to ~144 bytes (with notify/indicate property)
 * per descriptor: ~70 bytes


## Useful Links
 * [@lizardo](https://github.com/lizardo)'s [nRF8001 Experiments](https://github.com/lizardo/nrf8001)
   * used as a starting point to reverse engineer the proprietary setup message format for the chips
 * [@NordicSemiconductor](https://github.com/NordicSemiconductor)'s [ble-sdk-arduino](https://github.com/NordicSemiconductor/ble-sdk-arduino)
   * Original Arduino SDK for nRF8001
 * [@guanix](https://github.com/guanix)'s [arduino-nrf8001](https://github.com/guanix/arduino-nrf8001)
   * nRF8001 support for Arduino



