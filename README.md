ZJS API for Bluetooth Low Energy (BLE)
======================================

* [Introduction](#introduction)
* [Web IDL](#web-idl)
* [BLE-supported Events](#ble\-supported-events)
* [Class: BLE](#ble-api)
  * [ble.disconnect(address)](#bledisconnectaddress)
  * [ble.startAdvertising(name, uuids, [url])](#blestartadvertisingname-uuids-url)
  * [ble.stopAdvertising()](#blestopadvertising)
  * [ble.setServices(primaryServices)](#blesetservicesprimaryservices)
  * [ble.newPrimaryService(init)](#blenewprimaryserviceinit)
  * [ble.newCharacteristic(init)](#blenewcharacteristicinit)
  * [ble.newDescriptor(init)](#blenewdescriptorinit)
* [Class: Characteristic](#characteristic-api)
* [Supporting Objects](#supporting-objects)
  * [PrimaryServiceInit](#primaryserviceinit)
  * [CharacteristicInit](#characteristicinit)
  * [DescriptorInit](#descriptorinit)
* [Client Requirements](#client-requirements)
* [Sample Apps](#sample-apps)

Introduction
------------
The BLE API is based off the [bleno API](https://github.com/sandeepmistry/bleno). Bluetooth Low Energy (aka
[Bluetooth LE: Broadcast](https://www.bluetooth.com/what-is-bluetooth-technology/how-it-works/le-broadcast)) is a power-friendly version of Bluetooth
intended for IoT devices. It provides the ability to support low-bandwidth data
services to nearby devices.

A note about UUIDs. The BLE standard lets you have full 128-bit UUIDs or short
16-bit UUIDs (4 hex chars). We only support the short ones so far.

Based on the bleno API, these UUIDs are specified as a hexadecimal string, so
"2901" really means 0x2901. *NOTE: This seems like a bad practice and we should
perhaps require these numbers to be specified as "0x2901" instead, or else
treat them like decimals.*

Web IDL
-------

This IDL provides an overview of the interface; see below for documentation of
specific API functions.  We also have a short document explaining [ZJS WebIDL conventions](Notes_on_WebIDL.md).
<details>
<summary> Click to show/hide WebIDL</summary>
<pre>
// require returns a BLE object
// var ble = require('ble');
[ReturnFromRequire,ExternalInterface=(EventEmitter)]
interface BLE: EventEmitter {
    void disconnect(string address);
    void startAdvertising(string name, sequence < string > uuids, optional string url);
    void stopAdvertising();
    void setServices(sequence < PrimaryService > services);
    PrimaryService newPrimaryService(PrimaryServiceInit init);
    Characteristic newCharacteristic(CharacteristicInit init);
    DescriptorInit newDescriptor(DescriptorInit init);
};<p>
dictionary PrimaryServiceInit {
    string uuid;
    sequence < Characteristic > characteristics;
};<p>dictionary PrimaryService {
    string uuid;
    sequence < Characteristic > characteristics;
};<p>dictionary CharacteristicInit {
    string uuid;
    sequence < string > properties;                // 'read', 'write', 'notify'
    sequence < DescriptorInit > descriptors;
    ReadCallback onReadRequest;         // optional
    WriteCallback onWriteRequest;       // optional
    SubscribeCallback onSubscribe;      // optional
    UnsubscribeCallback onUnsubscribe;  // optional
    NotifyCallback onNotify;            // optional
};<p>interface Characteristic {
    attribute ReadCallback onReadRequest;
    attribute WriteCallback onWriteRequest;
    attribute SubscribeCallback onSubscribe;
    attribute UnsubscribeCallback onUnsubscribe;
    attribute NotifyCallback onNotify;
    attribute CharacteristicResult response;
};<p>callback ReadCallback = void (unsigned long offset,
                              FulfillReadCallback fulfillReadCallback);
[ExternalInterface=(Buffer)]
callback WriteCallback = void (Buffer data, unsigned long offset,
                               boolean withoutResponse,
                               FulfillWriteCallback fulfillWriteCallback);
callback SubscribeCallback = void (unsigned long maxValueSize,
                                   FulfillSubscribeCallback fullfillSubscribeCallback);
callback FulfillReadCallback = void (CharacteristicResult result, Buffer data);
callback FulfillWriteCallback = void (CharacteristicResult result);
callback FulfillSubscribeCallback = void (Buffer data);
callback NotifyCallback = void (any... params);
callback UnsubscribeCallback = void (any... params);
enum CharacteristicResult { "RESULT_SUCCESS", "RESULT_INVALID_OFFSET",
                            "RESULT_INVALID_ATTRIBUTE_LENGTH", "RESULT_UNLIKELY_ERROR" };
dictionary DescriptorInit {
    string uuid;
    string value;
};</pre>
</details>

BLE-supported Events
--------------------
BLE is an [EventEmitter](./events.md) with the following events:

### Event: `accept`

* `string` `clientAddress`

Emitted when a BLE client has connected. `clientAddress` is a unique BLE address
for the client in colon-separated format (e.g. 01:23:45:67:89:AB).

### Event: `advertisingStart`

* `int` `status`

Emitted when BLE services have begun to be advertised. The `status` will be 0
for success, otherwise for an error.

### Event: `disconnect`

* `string` `clientAddress`

Emitted when a BLE client has disconnected. `clientAddress` will be the same as
one previously sent with the `accept` event.

### Event: `stateChange`

* `string` `newState`

Emitted with `poweredOn` when the BLE stack is ready to be used. No other states
are supported at this time.

BLE API
-------
### ble.disconnect(address)
* `address` *string* The address of the connected client.

Disconnect the remote client.

### ble.startAdvertising(name, uuids, [url])
* `name` *string* The `name` is limited to 26 characters and will be
  advertised as the device name to nearby BLE devices.
* `uuids` *string[]*  The `uuids` array may contain at most 7 16-bit
  UUIDs (four hex digits each).  These UUIDs identify available
  services to nearby BLE devices.
* `url` *string* The `url` is optional and limited to around 24
  characters (slightly more if part of the URL is able to be
  [encoded](https://github.com/google/eddystone/tree/master/eddystone-url). If
  provided, this will be used to create a physical web advertisement
  that will direct users to the given URL. At that URL they might be
  able to interact with the advertising device somehow.

Advertises the name and url of the device.

### ble.stopAdvertising()

Currently does nothing.

### ble.setServices(primaryServices)
* `primaryServices` *array of [PrimaryService](#primaryservice) objects* The PrimaryService
  objects are used to set up the services that are implemented by your
  app.

The PrimaryService object contains the following fields:


### ble.newPrimaryService(init)
* `init` [*PrimaryServiceInit*](#primaryserviceinit)
* Returns: a new PrimaryService object.

### ble.newCharacteristic(init)
* `init` [*CharacteristicInit*](#characteristicinit)
* Returns: a new Characteristic object.


### ble.newDescriptor(init)
* `init` [*DescriptorInit*](#descriptorinit)
* Returns: a new DescriptorInit object.


Characteristic API
------------------
The "Characteristic" object contains the set of callbacks that...[[TODO!!!]]

Explanation of common arguments to the above functions:
* `offset` is a 0-based integer index into the data the characteristic
    represents.
* `result` is one of these values defined in the Characteristic object.
  * RESULT_SUCCESS
  * RESULT_INVALID_OFFSET
  * RESULT_INVALID_ATTRIBUTE_LENGTH
  * RESULT_UNLIKELY_ERROR
* `data` is a [Buffer](./buffer.md) object.

Supporting Objects
------------------

### PrimaryServiceInit

This object has two fields:
1. `uuid` *string* This field is a  16-bit service UUID (4 hex chars).
2. `characteristics` *array of [Characteristics](#characteristic-api)*


### CharacteristicInit

This object has 3 required fields:
1. `uuid` *string* This field is a 16-bit characteristic UUID (4 hex chars).
2. `properties` *array of strings* Possible values: 'read', 'write', and 'notify', depending on what is supported.
3. `descriptors` *array of [Descriptors](#descriptor)*

It may also contain these optional callback fields:
1. `onReadRequest` *ReadCallback*
  * Called when the client is requesting to read data from the characteristic.
  * See below for common argument definitions
2. `onWriteRequest` *WriteCallback*
  * Called when the client is requesting to write data to the characteristic.
  * `withoutResponse` is true if the client doesn't want a response
    * *TODO: verify this*
3. `onSubscribe` *SubscribeCallback*
  * Called when a client signs up to receive notify events when the
      characteristic changes.
  * `maxValueSize` is the maximum data size the client wants to receive.
4. `onUnsubscribe` *UnsubscribeCallback*
  * *NOTE: Never actually called currently.*
5. `onNotify` *NotifyCallback*
  * *NOTE: Never actually called currently.*


### DescriptorInit

This object has two fields:
1. `uuid` *string* This is a 16-bit descriptor UUID (4 hex chars)
    * Defined descriptors are listed here in [Bluetooth Specifications](https://www.bluetooth.com/specifications/gatt/descriptors)
2. `value` *string* This string supplies the defined information.
    * *NOTE: Values can also be Buffer objects, but that's not currently
    supported.*

Client Requirements
-------------------
You can use any device that has BLE support to connect to the Arduino 101 when
running any of the BLE apps or demos. We've successfully tested the following
setup:

* Update the Bluetooth firmware, follow instructions [here](https://wiki.zephyrproject.org/view/Arduino_101#Bluetooth_firmware_for_the_Arduino_101)
* Any Android device running Android 6.0 Marshmallow or higher (We use Nexus 5/5X, iOS devices not tested)
* Let the x86 core use 256K of flash space with ROM=256 as described [here](https://github.com/intel/zephyr.js#getting-more-space-on-your-arduino-101)

For the WebBluetooth Demo which supports the Physical Web, you'll need:
* Chromium version 50.0 or higher
* Make sure Bluetooth is on and Location Services is enabled
* Go to chrome://flags and enable the #enable-web-bluetooth flag

Sample Apps
-----------
* [BLE with multiple services](../samples/BLE.js)
* [WebBluetooth Demo](../samples/WebBluetoothDemo.js)
* [WebBluetooth Demo with Grove LCD](../samples/WebBluetoothGroveLcdDemo.js)
* [Heartrate Demo with Grove LCD](../samples/HeartRateDemo.js)
