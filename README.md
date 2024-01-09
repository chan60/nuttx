Apache NuttX is a real-time operating system (RTOS) with an emphasis on standards compliance and small footprint. Scalable from 8-bit to 64-bit microcontroller environments, the primary governing standards in NuttX are POSIX and ANSI standards. Additional standard APIs from Unix and other common RTOSs (such as VxWorks) are adopted for functionality not available under these standards, or for functionality that is not appropriate for deeply-embedded environments (such as fork()).

For brevity, many parts of the documentation will refer to Apache NuttX as simply NuttX.

## 시작하기

Nuttx는 마이크로칩셋에서 구동이 가능한 RTOS의 OS로 개발자에게 좀 더 친숙한 방식으로 개발이 가능함

<br/>

## 사용 보드 규격

- 보드 명칭 : NUCLEO-H743ZI2
- 보드 칩셋 : nucleo-h743zi2:nsh
- 메인 칩셋 : STM32H743
- 클럭 : 400 MHz
- Features
  - LQFP144 package
  - 3 user LETs
  - 32.768 kHz Crystal oscilator
  - ST-LINK USB V_bus,
  - Ethernet (RJ45)

<br/>

## 설치 (macOS)

1. Package install

```bash
brew tap discoteq/discoteq && brew install flock && brew install x86_64-elf-gcc && brew install u-boot-tools
```

2. Install kconfig frontend

```bash
git clone https://bitbucket.org/nuttx/tools.git && cd tools/kconfig-frontends && patch < ../kconfig-macos.diff -p 1 && ./configure --enable-mconf --disable-shared --enable-static --disable-gconf --disable-qconf --disable-nconf && make && sudo make install
```

3. Install toolchain for STM32 (32 bit ARM)

```bash
brew install --cask gcc-arm-embedded
```

4. Clone repository

```bash
mkdir nuttxspace
cd nuttxspace
git clone git@github.com:chan60/nuttx
git clone git@github.com:chan60/apps
```

<br/>
<br/>

## 초기 설정

1. 설정 가능한 보드 리스트 불러오기 `~/github_repos/nuttxspace/nuttx`

```bash
./tools/configure.sh -L | less
```

2. 환경 및 보드 설정하기

```bash
./tools/configure.sh -m nucleo-h743zi2:nsh
```

3. 설정 바꾸기

```bash
make menuconfig
```

- 설정 초기화

```bash
make distclean
```

1. 빌드하기

```bash
make
```

</br>
</br>

## 플래싱

1. 플래싱 앱 `openocd` 설치

```bash
brew install openocd
```

2. 보드 연결 (usb)

3. 플래싱

```bash
cd ~/github_repos/nuttxspace/nuttx

openocd -f interface/stlink.cfg -f target/stm32h7x.cfg -c 'init' \
  -c 'program nuttx.hex verify reset' -c 'shutdown'
```

<br/>
<br/>

## 디버깅

### Enable debug features in menu

1. Engage into menuconfig `make menuconfig`
2. Enter `Build Setup > Debug Options`
3. Set check below features
   1. `Enable Debug Features` - selecting this will turn on subsystem-level debugging options, they will become visible on the page below. You can then select the ones you want.
   2. `Enable Error Output` - this will only log errors.
   3. `Enable Warnings Output` — this will log warnings and errors.
   4. `Enable Informational Debug Output` — this will produce informational output, warnings, and errors.
4. Save to `.config` with `s` key

- All things can edit also in `nuttx/.config` like below things

```conf
CONFIG_DEBUG_FEATURES=y
CONFIG_DEBUG_ERROR=y
CONFIG_DEBUG_WARN=y
CONFIG_DEBUG_INFO=y
```

<br/>

### Debugging with `openocd` and `gdb`

1. Enter debug mode with `openocd`

```bash
cd nuttx/
openocd -f interface/stlink.cfg -f target/stm32h7x.cfg
```

2. Enter & connect board with `gdb`

```bash
cd nuttx/
gdb
target extended-remote :3333
```

- `gdb` commands
  - to reset the board

```bash
mon reset
```

- to halt the board

```bash
mon halt
```

- breakpoint nsh*main *## This does not work my time\_

```bash
breakpoint nsh_main
```

- finally start nuttx

```bash
continue
```

</br>
</br>

# Errors

### Verify failed with timeout

- Official code가 버전에 맞게 업데이트되지 않은 것으로 보임 아래 코드는 사용하면 플래시가 되지 않음 상부 매뉴얼 참조할 것

```bash
openocd -f interface/stlink-v2.cfg -f target/stm32f1x.cfg -c 'init' \
  -c 'program nuttx/nuttx.bin verify reset' -c 'shutdown'
```

- deprecated things
  - `stlink-v2.cfg ► stlink.cfg`
- nucleo-h743zi2는 stm32h7x 시리즈임
  - `stm32f1x.cfg ► stm32h7x.cfg`
- upload file issue
  - `nuttx/nuttx.bin ► nuttx.hex`

### VS Code **IncludePath Error**

- 각각의 리포지토리에 `.vscode/c_cpp_properties.json`파일 내 includePath에 참조 경로를 추가할 것

in `nuttxspace/nuttx`

```json
{
  "configurations": [
    {
      "name": "Mac",
      "includePath": ["${default}", "${workspaceFolder}/include/", "${workspaceFolder}/../apps/**"],
      "defines": [],
      "macFrameworkPath": ["/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/System/Library/Frameworks"],
      "cStandard": "c17",
      "cppStandard": "c++17",
      "intelliSenseMode": "macos-clang-x64"
    }
  ],
  "version": 4
}
```

in `nuttxspace/apps`

```json
{
  "configurations": [
    {
      "name": "Mac",
      "includePath": ["${workspaceFolder}/**", "${workspaceFolder}/../nuttx/include/"],
      "defines": [],
      "macFrameworkPath": ["/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/System/Library/Frameworks"],
      "compilerPath": "/usr/bin/clang",
      "cStandard": "c17",
      "cppStandard": "c++17",
      "intelliSenseMode": "macos-clang-x64"
    }
  ],
  "version": 4
}
```
