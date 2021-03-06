upb-cli [![Gitter chat](https://img.shields.io/gitter/room/DaAwesomeP/upb-cli.js.svg?maxAge=2592000&style=flat-square)](https://gitter.im/DaAwesomeP/upb-cli)
=======
[![npm](http://img.shields.io/npm/v/upb-cli.svg?style=flat-square)](https://www.npmjs.org/package/upb-cli) [![npm](http://img.shields.io/npm/dm/upb-cli.svg?style=flat-square)](https://www.npmjs.org/package/upb-cli) [![David](https://img.shields.io/david/DaAwesomeP/upb-cli.svg?style=flat-square)](https://david-dm.org/DaAwesomeP/upb-cli) [![npm](http://img.shields.io/npm/l/upb-cli.svg?style=flat-square)](https://github.com/DaAwesomeP/upb-cli/blob/master/LICENSE)
---

DEPRECATED
==========
### Please use the [upb](https://github.com/DaAwesomeP/node-upb) package programmatically instead [(https://github.com/DaAwesomeP/node-upb)](https://github.com/DaAwesomeP/node-upb). This CLI wrapper package is no longer being maintained.

A CLI interface to the UPB library that generates decodes UPB (Universal Powerline Bus) commands. **If you are looking for the NodeJS library version of this, then please see [node-upb](https://github.com/DaAwesomeP/node-upb/).**

**This has only been tested with Simply Automated switches!** While probably won't harm your device (it is sending control commands, not core commands), I'm not sure what will happen when this is used with other branded switches (like PCS).

## Installation
Install it globally:
```bash
$ npm install -g upb-cli
```
Or install it locally:
```bash
$ npm install upb-cli
```

## Usage

**If you plan on making your own serial implementation using this, remember to put the PIM in message mode first and to proceed each UPB command with the ASCII #20 character and to end it with the ASCII #13 character.**

```bash
$ upb-cli [options]
```

Output of `--help`:
```bash
$ upb-cli --help

  Usage: upb-cli [options]

  Options:

    -h, --help                         output usage information
    -V, --version                      output the version number
    -e, --generate                     Generates a UPB command and outputs the command or JSON. Use with all other arguments but --decode and --listCommands. This is on by default (for backwards compatibility with versions 1.0.x) unless --decode or --listCommands are specified.
    -d, --decode [command]             Decodes a UPB command and outputs JSON. Ignores all other commands but --jsonStringify. Use with all other arguments but generate and listCommands THIS WILL NOT ERVIFY THE CHECKSUM!
    --send                             Set whether to actually send the command or just to generate it. Defaults to false.
    --debug                            Set whether to output serial information to the console when using --send. Defaults to false.
    --json                             Set whether to respond with the JSON array instead of just the command. Defaults to false.
    --jsonStringify                    Set whether to stringify the JSON array response. When generating, this only applies if --json is present. Otherwise ignored. When decoding, --json is not required. Defaults to false.
    -p, --port [port]                  Set serial port
    -o, --source [id (number)]         Set PIM source ID. Defaults to 255, which is almost always fine.
    -n, --net [id (number)]            Set Network ID. Use 0 for the global network (controls all devices).
    -i, --id [number]                  Set link or device ID
    -t, --type [link or device]        Set whether to control a link or device
    -x, --sendx [number]               Set the number of times to send the command. Accepts numbers 1 through 4. Defaults to 1.
    -s, --sendTime [number]            Send the number of time this command is sent out of the total (sendx). NOTE: THE PIM WILL AUTOMATICALLY SEND THE CORRECT NUMBER OF COMMANDS! So, this is only useful for displaying commands and not sending them. Accepts numbers 1 through 4. Cannot be greater than sendx. Defaults to 1.
    --ackPulse                         Request an acknowledge pulse. Defaults to false.
    --idPulse                          Request an ID pulse. Defaults to false.
    --ackMsg                           Request an acknowledge message. Defaults to false.
    -e, --powerlineRepeater [1, 2, 4]  Request for the command to go through a powerline repeater. Set or numbers 1, 2, 4, or false. Defaults to false.
    -c, --cmd [command]                Set the command to send. You may also use the command numbers associated with those commands. See --listCommands.
    -m, --listCommands                 Lists all available commands.
    -l, --level [percent]              Set the level. Only applies to goto, fadeStart, fadeStop, and toggle. Accepts values 0 through 100. Otherwise this will be ignored. Required with goto and fade start.
    -r, --rate [seconds]               Set the fade rate. Use false for instant on. Only applies to goto, fadeStart, and toggle. Otherwise  this will be ignored. Defaults to device settings.
    -h, --channel [number]             Set the channel to use. Use false for default. Only applies to goto, fadeStart, blink, and toggle. Otherwise this will be ignored. Only works on some devices. Defaults to off (command not sent).
    -b, --blinkRate [number]           Set the blink rate. USE CAUTION WITH LOW NUMBERS! I am not sure what unit this is in. Accepts values 1 through 255. Required for blink. Only applies to blink. Otherwise this will be ignored.
    -g, --toggleCount [number]         Set the toggle count. Only applies to toggle. Otherwise this will be ignored. Required for toggle.
    -a, --toggleRate [seconds]         Set the toggle rate. Only applies to toggle. Otherwise this will be ignored. Defaults to 0.5.
```

## Examples

1. Generate: Device #4 on network 21 go to 75% brightness in 3 seconds, send this command 2 times, and output full JSON:
   
   ```bash
   $ upb-cli --generate --net 21 --id 4 --type device --cmd goto --level 75 --rate 3 --sendx 2 --json
   
   { source: 255,
     sendx: '2',
     ackPulse: false,
     idPulse: false,
     ackMsg: false,
     powerlineRepeater: false,
     action: 'generate',
     sendTime: 1,
     network: '21',
     id: '4',
     type: 'device',
     cmd: 'goto',
     level: '75',
     rate: '3',
     ctrlWord: { byte1: 0, byte2: 9, byte3: 0, byte4: 4 },
     words: 9,
     hex:
      { network: '15',
        id: '4',
        source: 'ff',
        msg: '22',
        level: '4b',
        rate: '3',
        ctrlWord:
         { byte1: '0',
           byte2: '9',
           byte3: '0',
           byte4: '4',
           fullByte1: '09',
           fullByte2: '04' } },
     msg: 22,
     generated: '09041504FF224B036B',
     checksum: '6b' }
   ```
   
2. Generate: Activate link #5 on network 0 (all devices) and send to serial port COM1 (Windows serial port with PIM connected):
   
   ```bash
   $ upb-cli -e -n 0 -i 5 -t link -c 20 --send -p COM1
   87000005FF2055
   ```
   
3. Generate: Toggle (like blink but with a maximum amount of times to blink) device #3 on network 67 six times every 5 seconds and send to serial port with debug messages /dev/ttyS0 (Linux/Mac OSX serial port with PIM connected) and output stringified JSON:
   
   ```bash
   $ upb-cli -e -n 67 -i 3 -t device -c toggle -g 6 -a 5 --json --jsonStringify --debug --send -p /dev/ttyS0
   {"source":255,"sendx":1,"ackPulse":false,"idPulse":false,"ackMsg":false,"powerlineRepeater":false,"action":"generate","sendTime":1,"network":"67","id":"3","type":"device","cmd":"toggle","toggleCount":"6","toggleRate":"5","ctrlWord":{"byte1":0,"byte2":9,"byte3":0,"byte4":0},"words":9,"hex":{"network":"43","id":"3","source":"ff","msg":"27","toggleCount":"6","toggleRate":"5","ctrlWord":{"byte1":"0","byte2":"9","byte3":"0","byte4":"0","fullByte1":"09","fullByte2":"00"}},"msg":27,"generated":"09004303FF27060580","checksum":"80"}
   
   Serial Port Opened
   
   Serial write RAW: ↨70028E
   Serial write encoded: 70028E
   Serial results: 8
   
   Serial write RAW: ¶09004303FF27060580
   Serial write encoded: 09004303FF27060580
   Serial results: 20
   
   Serial Port Closed
   ```
   
4. Decode: 08008708FF2264E
   
   ```bash
   $ upb-cli -d 08008708FF2264E
   { source: 255,
     sendx: 1,
     sendTime: 1,
     ackPulse: false,
     idPulse: false,
     ackMsg: false,
     powerlineRepeater: 0,
     action: 'decode',
     generated: '08008708FF2264E4',
     hex:
      { ctrlWord:
         { fullByte1: '08',
           fullByte2: '00',
           byte1: '0',
           byte2: '8',
           byte3: '0',
           byte4: '0' },
        network: '87',
        id: '08',
        source: 'FF',
        msg: '22',
        level: '64' },
     ctrlWord: { byte1: 0, byte2: 8, byte3: 0, byte4: 0 },
     type: 'device',
     words: 8,
     network: 135,
     id: 8,
     msg: '22',
     cmd: 'goto',
     checksum: 'E4',
     level: 100 }
   ```

## More Information

I got most of the information the last three items listed on this Simply Automated page: [Tech Specs](http://www.simply-automated.com/tech_specs/). I also experimented with my serial terminal to see responses of other switches.

 - **UPB System Description** - This PDF describes all parts of the UPB protocol.
 - **UPB Command Wizard - Software** - This program lets you build commands with a wizard/GUI and see the result. It does not actually send the command, but it is very valuable for understanding the commands without reading too much of the above PDF.
 - **UPB Powerline Interface Module (PIM) - Description** - This PDF contains information about the PIM. It shows serial specifications (4800 baud 8-n-1) and PIM responses. It look me a while to figure out that the PIM always responds with `PE` whenever a command is not prefixed by the #20 character.
