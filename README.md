<h2>SwiftyLinkerKit
  <img src="http://zeezide.com/img/LKDigi128.png"
       align="right" width="128" height="128" />
</h2>

![Swift4](https://img.shields.io/badge/swift-4-blue.svg)
![tuxOS](https://img.shields.io/badge/os-tuxOS-green.svg?style=flat)
<a href="https://slackpass.io/swift-arm"><img src="https://img.shields.io/badge/Slack-swift/arm-red.svg?style=flat"/></a>
<a href="https://travis-ci.org/SwiftyLinkerKit/SwiftyLinkerKit"><img src="https://travis-ci.org/SwiftyLinkerKit/SwiftyLinkerKit.svg?branch=develop" /></a>

A 
[Swift](http://swift.org)
module to control
[LinkerKit](http://www.linkerkit.de/)
components in a convenient way.
This is currently focused on LinkerKit things attached to a
Raspberry Pi
[LK-Base-RB 2](http://www.linkerkit.de/index.php?title=LK-Base-RB_2)
shield (since this is what I have 🤓).

<a href="http://www.linkerkit.de"><img width="256" align="center" src="http://www.linkerkit.de/images/f/f8/linkerkitWiki.png" /></a>

**SwiftyLinkerKit** is based on the excellent
[SwiftyGPIO](https://github.com/uraimo/SwiftyGPIO)
module, and of course on the great work of the
[Swift-ARM Community](https://slackpass.io/swift-arm).

## Supported Components

Right now SwiftyLinkerKit supports only a few components listed below.
It is very easy to add new ones, we very much welcome contributions!

Also: We need help with understanding LK-Accel and LK-Temp (how to connect them,
how to hook them up w/ SPI, [contact us](#Who) if you know more about that
weird hardware stuff 🤖).

### LK-Button 2

<div style="display: block; margin: 0 auto;">
  <a href="http://www.linkerkit.de/index.php?title=LK-Button2"><img align="right" width="128" src="http://www.linkerkit.de/images/1/11/lk_button2.png" /></a>
</div>

Two buttons :-)

```swift
import SwiftyLinkerKit

let shield  = LKRBShield.default
let buttons = LKButton2()

shield.connect(buttons, to: .digital2122)

buttons.onPress1 {
    print("Button 1 was pressed!")
}
buttons.onChange2 { isPressed in
    print("Button 2 changed, it is now: \(isPressed ? "pressed" : "off" )")
}
```

### LK-Digi

<a href="http://www.linkerkit.de/index.php?title=LK-Digi"><img align="right" width="128" src="http://www.linkerkit.de/images/thumb/8/83/LK-Digi.jpg/358px-LK-Digi.jpg.png" /></a>

A neat 7-segment display.

```swift
import SwiftyLinkerKit

let shield  = LKRBShield.default
let display = LKDigi()

shield.connect(display, to: .digital45)

display.show("SWIFT")
sleep(2)

display.show(1337)
sleep(2)

display.showTime()
sleep(2)

for i in (0...10).reversed {
    display.show(i)
    sleep(1)
}
```

### LK-PIR

<a href="http://www.linkerkit.de/index.php?title=LK-PIR"><img align="right" width="128" src="http://www.linkerkit.de/images/thumb/7/78/LK-PIR-01.jpg/358px-LK-PIR-01.jpg" /></a>

An IR movement detector.

```swift
import SwiftyLinkerKit

let shield   = LKRBShield.default
let watchdog = LKPIR()

shield.connect(watchdog, to: .digital1213)

watchdog.onChange { didMove in
    if didMove { print("careful, don't move!") }
    else       { print("nothing is moving.")   }
}
```

### LK-Temp

<a href="http://www.linkerkit.de/index.php?title=LK-Temp"><img align="right" width="128" src="http://www.linkerkit.de/images/thumb/6/6a/LK-Temp.png/358px-LK-Temp.png" /></a>

Temperature sensor, uses a thermistor to detect the environmental temperature.
LK-Temp is connected to one of the *analog* ports (and hosted via the board ADC
running on SPI).

```swift
import SwiftyLinkerKit

let shield      = LKRBShield.default
let thermometer = LKTemp(interval: 5.0, valueType: .celsius)

thermometer.onChange { temperature in
    print("Temperatur is", temperature, "℃")
}
```


## How to setup and run

Note: This is for 32-bit, 64-bit doesn't seem to work yet.

### Raspi Docker Setup

You don't have to, but I recommend running things in a
[HypriotOS](https://blog.hypriot.com/post/releasing-HypriotOS-1-8/)
docker container.

Setup is trivial. Grab the [flash](https://github.com/hypriot/flash) tool,
then insert your empty SD card into your Mac
and do:
```shell
$ flash --hostname zpi3 \
  https://github.com/hypriot/image-builder-rpi/releases/download/v1.8.0/hypriotos-rpi-v1.8.0.img.zip
```

Boot your Raspi and you should be able to reach it via `zpi3.local`.

I also recommend to use docker-machine (e.g. see 
[here](https://github.com/helje5/dockSwiftOnARM/wiki/Remote-Control-Raspi-Docker)),
but that is not necessary either.

### Running an ARM Swift container

Boot the container like so:

```shell
$ docker run --rm \
  --cap-add SYS_RAWIO \
  --privileged \
  --device /dev/mem \
  -it --name swiftfun \
  helje5/rpi-swift-dev:4.1.0 /bin/bash
```

You end up in a Swift 4.1 environment with some dev tools like emacs 
pre-installed. Sudo password for user `swift` is `swift`.

### Importing the Swift Package

Sample `Package.swift` file:

```swift
// swift-tools-version:4.0

import PackageDescription

let package = Package(
    name: "testit",
    dependencies: [
        .package(url: "https://github.com/SwiftyLinkerKit/SwiftyLinkerKit.git",
                 from: "0.1.0"),
    ],
    targets: [
        .target(
            name: "testit",
            dependencies: [ "SwiftyLinkerKit" ]),
    ]
)
```


## Example: Clock

A simple digital clock.

```shell
swift@f296eaf9ee96:~$ mkdir clock && cd clock && swift package init --type executable
Creating executable package: clock
Creating Package.swift
Creating README.md
Creating .gitignore
Creating Sources/
Creating Sources/clock/main.swift
Creating Tests/
```

Then edit the `Package.swift` file to look like this:
```swift
// swift-tools-version:4.0

import PackageDescription

let package = Package(
    name: "clock",
    dependencies: [
        .package(url: "https://github.com/SwiftyLinkerKit/SwiftyLinkerKit.git",
                 from: "0.1.0"),
    ],
    targets: [
        .target(
            name: "clock",
            dependencies: [ "SwiftyLinkerKit" ]),
    ]
)
```

Edit the `Sources/clock/main.swift` with the following Swift code. In the
example the LK-Digi is connected to the Digital-4/5 slot of the LK-RB-Shield,
adjust accordingly.

```swift
import SwiftyLinkerKit
import Dispatch

let shield  = LKRBShield.default
let display = LKDigi()

shield.connect(display, to: .digital45)

let timer = DispatchSource.makeTimerSource()

timer.setEventHandler {
    display.showTime()
}

timer.schedule(deadline  : .now(),
               repeating : .seconds(1),
               leeway    : .milliseconds(1))
timer.resume()

dispatchMain()
```

Build everything:
```shell
swift@f296eaf9ee96:~/testit$ swift build
Fetching https://github.com/SwiftyLinkerKit/SwiftyLinkerKit.git
Fetching https://github.com/uraimo/SwiftyGPIO.git
Fetching https://github.com/AlwaysRightInstitute/SwiftyTM1637.git
Cloning https://github.com/SwiftyLinkerKit/SwiftyLinkerKit.git
Resolving https://github.com/SwiftyLinkerKit/SwiftyLinkerKit.git at 0.1.0
Cloning https://github.com/uraimo/SwiftyGPIO.git
Resolving https://github.com/uraimo/SwiftyGPIO.git at 1.0.5
Cloning https://github.com/AlwaysRightInstitute/SwiftyTM1637.git
Resolving https://github.com/AlwaysRightInstitute/SwiftyTM1637.git at 0.1.2
Compile Swift Module 'SwiftyGPIO' (10 sources)
Compile Swift Module 'SwiftyTM1637' (5 sources)
Compile Swift Module 'SwiftyLinkerKit' (5 sources)
Compile Swift Module 'clock' (1 sources)
Linking /home/swift/clock/.build/armv7-unknown-linux-gnueabihf/debug/clock
```

You need to run it using `sudo` (password in the Docker is `swift`):
```shell
swift@f296eaf9ee96:~/testit$ sudo .build/armv7-unknown-linux-gnueabihf/debug/clock
```



Want to see it in action?
<a href="https://twitter.com/helje5/status/1004022796924674048">SwiftyLinkerKit driven input/output using LinkerKit components</a>


### Who

**SwiftyLinkerKit** is brought to you by
[AlwaysRightInstitute](http://www.alwaysrightinstitute.com).
We like feedback, GitHub stars, 
cool [contract work](http://zeezide.com/en/services/services.html),
presumably any form of praise you can think of.

There is a channel on the [Swift-ARM Slack](http://swift-arm.noze.io).
