# petoneer-smartdot-bluetooth-protocol

The Petoneer Smartdot Bluetooth protocol reverse-engineered.

I've built a simple webapp to play the pre-set games - check it out [here](https://smartdot.kolombo.lv) - [repo](https://github.com/marcomow/petoneer-smartdot-webapp).

## Protocol

Device name: `Petcat`.

Service: `0xfff0`.

Characteristic: `0xfff3`.

Pre-set games commands (values):

- small: `0f0405000107`;
- medium: `0f0405000208`;
- large: `0f0405000309`.

> NB there's also the possibility to send "manual" commands where you can specify the exact position of the red light, but I'm still working on reverse engineering it.

## The protocol in action

These are extracts taken from the webapp as an example and to help understand how to use the protocol.

### Connection

The Smartdot I have is easy to connect, filtering by the advertised name of the device - in my case `PetCat` - any by the provided optional services - 0xfff0.

```javascript
const device = await navigator.bluetooth.requestDevice({
                filters: [
                    { name: 'PetCat' }
                ],
                optionalServices: [0xfff0]
            });
const server = await this.device.gatt?.connect?.();
```

### Triggering pre-set games


```javascript
const setPresetCommand = (server, command) => {
        const service = await server.getPrimaryService(0xfff0);
        const characteristic = await service.getCharacteristic(0xfff3);
        const commandValue = command.match(/.{1,2}/g);
        const value = commandValue.map(v => parseInt(v, 16));
        characteristic.writeValue(new Uint8Array(value));
    }

```

## Motivation

I've bought a Petoneer Smartdot to automate playing with my cats using Home Assistant, in particular when I'm not home.

It didn't came with an integration, so I searched for a way to build it myself, but as my knowledge of Python is very basic and I feel way more comfortable with Javascript/Typescript I've built an add-on. That was quite doing the job, but I built it somehow in a wrong way and it was continuously connecting to the Smartdot, thus blocking the Home Assistant bluetooth entirely.

After some months the new Home Assistant Bluetooth integration came out, so I thought I could finally build the integration by myself, but lacking time and knowledge of Python, I did not work on it yet.

Therefore, as suggested by [@blackadder](https://github.com/blakadder) (thank you!), I am documenting here the results of my reverse engineering of the protocol, although it's not yet clear to me how the "manual" mode works.
