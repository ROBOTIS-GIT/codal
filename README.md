# Codal [![Build Status](https://travis-ci.org/lancaster-university/codal.svg?branch=master)](https://travis-ci.org/lancaster-university/codal)

The main repository for the Component Oriented Device Abstraction Layer (CODAL).

This repository is an empty shell that provides the tooling needed to produce a bin/hex/uf2 file for a CODAL device.

## ROBOTIS 개발자를 위한 문서
Fork 이전의 repo는 codal의 빌드시스템을 위한 저장소였다.

codal은 json파일을 통해 의존성 패키지 설치를 진행하는데, 이 방식을 보다 간소하게 하여 우리 개발자가 개발환경 구축을 보다 쉽게 하기 위해 별도의 저장소를 구축하였다.

기존의 json에서 처리하는 것 대신에, submodule을 통해 개발자가 이 빌드시스템을 git으로 다운받으면, 자동으로 의존성 패키지들의 repos와 연결하도록 설정하였다.

따라서, 개발자는 submodule에 있는 각 패키지들을 원하는 버전으로 checkout하여 사용하거나 branch생성을 포함한 git을 통한 개발을 진행하면 된다.

즉, 추가된 폴더/파일 구조는 아래와 같다.

- **libraries**
    - **codal-cm300** : 타겟보드 패키지
    - **codal-core** : codal core 패키지
    - **codal-nrf52** : codal nrf52 core 패키지
- **source**
    - **main.cpp** : 빌드시 사용되는 어플리케이션 코드(main함수)
- **hex2uf2.bat** : 빌드된 HEX파일을 uf2파일로 변환하는 스크립트 파일(batch) for only WINDOWS

개발환경 구축을 스텝별로 설명하면 아래와 같다.

1. 빌드시스템에 필요한 디펜던시 설치 : [Installation 참조](#Installation)
2. codal git clone
```bash
$ git clone --recursive https://github.com/ROBOTIS-GIT/codal.git -b develop
```
3. 각 라이브러리 패키지에 대해 원하는 커밋/브랜치로 checkout (git checkout commit/branch)
4. main.cpp 수정 및 빌드 : [Building 참조](#Building)


## Installation

### Automatic installation.

This software has its grounding on the founding principles of [Yotta](https://www.mbed.com/en/platform/software/mbed-yotta/), the simplest install path would be to install their tools via their handy installer.

### Docker

A [docker image](https://hub.docker.com/r/jamesadevine/codal-toolchains/) is available that contains toolchains used to build codal targets. A wrapper [Dockerfile](https://github.com/lancaster-university/codal-docker) is available that can be used to build your project with ease.

Then follow the build steps listed below.

### Manual installation

1. Install `git`, ensure it is available on your platforms path.
2. Install the `arm-none-eabi-*` command line utilities for ARM based devices and/or `avr-gcc`, `avr-binutils`, `avr-libc` for AVR based devices, ensure they are available on your platforms path.
3. Install [CMake](https://cmake.org)(Cross platform make), this is the entirety of the build system.
    5. If on Windows, install ninja.
4. Install `Python 2.7` (if you are unfamiliar with CMake), python scripts are used to simplify the build process.
5. Clone this repository

# Building
- Generate or create a `codal.json` file
    - `python build.py ls` lists all available targets
    - `python build.py <target-name>` generates a codal.json file for a given target
- In the root of this repository type `python build.py` the `-c` option cleans before building.
    - If you are not using python:
        - Windows:
            1. In the root of the repository make a build folder.
            2. `cd build`
            3. `cmake .. -G "Ninja" -DCMAKE_BUILD_TYPE=RelWithDebInfo`
            4. `ninja`
        - Mac:
            1. In the root of the repository make a build folder.
            2. `cd build`
            3. `cmake .. -G "Unix Makefiles" -DCMAKE_BUILD_TYPE=RelWithDebInfo`
            4. `make`

- The hex file will be placed at the location specified by `codal.json`, by default this is the root.

# Configuration

Below is an example of how to configure codal to build the [codal-circuit-playground](https://github.com/lancaster-university/codal-circuit-playground) target, example applications will automatically be loaded into the "source" folder:

```json
{
    "target":{
        "name":"codal-circuit-playground",
        "url":"https://github.com/lancaster-university/codal-circuit-playground",
        "branch":"master",
        "type":"git"
    }
}
```

For more targets, read the targets section below.

## Advanced

If you would like to override or define any additional configuration options (`#define's`) that are used by the supporting libraries, the codal build system allows the addition of a config field in `codal.json`:

```json
{
    "target":{
        "name":"codal-circuit-playground",
        "url":"https://github.com/lancaster-university/codal-circuit-playground",
        "branch":"master",
        "type":"git"
    },
    "config":{
        "NUMBER_ONE":1
    },
    "application":"source",
    "output_folder":"."
}
```

The above example will be translate `"NUMBER_ONE":1` into: `#define NUMBER_ONE     1` and force include it during compilation. You can also specify alternate application or output folders.

# Targets

To obtain a full list of targets type:

```
python build.py ls
```

To generate the `codal.json` for a target listed by the ls command, please run:

```
python build.py <target-name>
```

Please note you may need to remove the libraries folder if your previous build relied on similar dependencies.

## Arduino Uno

This target specifies the arduino uno which is driven by an atmega328p.

### codal.json specification
```json
"target":{
    "name":"codal-arduino-uno",
    "url":"https://github.com/lancaster-university/codal-arduino-uno",
    "branch":"master",
    "type":"git"
}
```
This target depends on:
* [codal-core](https://github.com/lancaster-university/codal-core) provides the core CODAL abstractions
* [codal-atmega328p](https://github.com/lancaster-university/codal-atmega328p) implements basic CODAL components (I2C, Pin, Serial, Timer)

## BrainPad

This target specifies the BrainPad which is driven by a STM32F.

### codal.json specification
```json
"target":{
    "name":"codal-brainpad",
    "url":"https://github.com/lancaster-university/codal-brainpad",
    "branch":"master",
    "type":"git"
}
```
This target depends on:
* [codal-core](https://github.com/lancaster-university/codal-core) provides the core CODAL abstractions
* [codal-mbedos](https://github.com/lancaster-university/codal-mbed) implements required CODAL basic components (Timer, Serial, Pin, I2C, ...) using Mbed

## Circuit Playground

This target specifies the circuit playground which is driven by a SAMD21.

### codal.json specification
```json
"target":{
    "name":"codal-circuit-playground",
    "url":"https://github.com/lancaster-university/codal-circuit-playground",
    "branch":"master",
    "type":"git"
}
```
This target depends on:
* [codal-core](https://github.com/lancaster-university/codal-core) provides the core CODAL abstractions
* [codal-mbed](https://github.com/lancaster-university/codal-mbed) implements required CODAL basic components (Timer, Serial, Pin, I2C, ...) using Mbed
* [codal-samd21](https://github.com/lancaster-university/codal-samd21) implements SAMD21-specific components (such as USB)
* [mbed-classic](https://github.com/lancaster-university/mbed-classic) is a fork of mbed, used by codal-mbed
