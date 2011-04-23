
Arduino CMake
=============

Arduino is a great development platform, which is easy to use. It has everything a beginner should need. The Arduino IDE simplifies a lot of things for the standard user, but if you are a professional programmer the IDE can feel simplistic and restrictive.

One major drawback of the Arduino IDE is that you cannot do anything without it, which for me is a complete buzz kill. Thats why I created an alternative build system for the Arduino using CMake.

CMake is great corss-platform build system that works on practically any operating system. With it you are not constrained to a single build system. CMake lets you generated the build system that fits your needs, using the tools you like. It can generate any type of build system, from simple Makefiles, to complete projects for Eclipse, Visual Studio, XCode, etc.

The Arduino CMake build system integrates tightly with the Arduino SDK. I'm currently basing on version 0022 of the Arduino SDK.

Requirements:

    CMake - http://www.cmake.org/cmake/resources/software.html
    Arduino SDK - http://www.arduino.cc/en/Main/Software


Linux Requirements:

    gcc-avr      - AVR GNU GCC compiler
    binutils-avr - AVR binary tools
    avr-libc     - AVR C library
    avrdude      - Firmware uploader


TODO:

    - Sketch conversion (PDE files)
    - Make work on, mainly dependency detection:
        - Windows
        - Mac OS X
    - Test more complex configurations
    - More robust error handling

Contents
========

    1. Getting Started
    2. Creating a firmware image
    3. Defining libraries

1. Getting Started
==================

The following inscructions are for *nix type systems, specifically this is a Linux example.

In short you can get up and running using the follwoing commands:

    mkdir build
    cd build
    cmake ..
    make
    make upload                      # to upload the firmware   [optional]
    make wiere_master_reader-serial  # to get a serial terminal [optional]

For a more detailed explanation, please read on...

1. Toolchain file

In order to build firmware for the Arduino you have to specify a toolchain file to enable cross-compilation. There are two ways of specifying the file, either at the command line or through the CMake cache. The bundled example configuration uses the second approche like so:

    set(CMAKE_TOOLCHAIN_FILE ${CMAKE_SOURCE_DIR}/cmake/toolchains/Arduino.cmake)

If you would like to specify it from the command line, heres how:

    cmake -DCMAKE_TOOLCHAIN_FILE=../path/to/toolchain/file.cmake PATH_TO_SOURCE_DIR

2. Creating a build directory

The second order of buissnes is creating a build directory. CMake has a great feature called out-of-source builds, what this means is the building is done in a completely separate directory, than where the sources are. The benefits of this is you don't have any clutter in you source directory and you won't accidentally commit something in, that is auto-generated.

So lets create that build directory:

    mkdir build
    cd build

3. Creating the build system

Now lets create the build system that will create our firmware:

    cmake ..

4. Building

Next we will build everything:

    make

5. Uploading

Once everything built correctly we can upload. Depending on your Arduino you will have to update the serial port used for uploading the firmware. To change the port please edit the following variable in CMakeLists.txt:

    set(${FIRMWARE_NAME}_PORT /path/to/device)

Ok lets do a upload:

    make upload

6. Serial output

If you have some serial output, you can launch a serial terminal from the build system. The command used for executing the serial terminal is user configurable by the following setting:

    set(${FIRMWARE_NAME}_SERIAL serial command goes here)

In order to get access to the serial port use the following in your command:

    @INPUT_PORT@

That constant will get replaced with the actual serial port used (see uploading). In the case of our example configuration we can get the serial terminal by executing the following:

    make wire_master_reader-serial







2. Creating a firmware image
============================


The first step in generating Arduino firmware is including the Arduino CMake module package. This easily done with:

    find_package(Arduino)

To have a specific minimal version of the Arduino SDK, you can specify the version like so:

    find_package(Arduino 22)

That will require an Arduino SDK version 0022 or newer. To ensure that the SDK is detected you can add the REQUIRED keyword:


    find_package(Arduino 22 REQUIRED)

Once you have the Arduino CMake package loaded you can start defining firmware images.

To create Arduino firmware in CMake you use the generate_arduino_firmware command. This function only accepts a single argument, the target name. To configure the target you need to specify a list of variables of the following format before the command:

    ${TARGET_NAME}${OPTION_SUFFIX}

Where ${TARGET_NAME} is the name of you target and ${OPTIONS_SUFFIX} is one of the following option suffixes:

     _SRCS           # Target source files
     _HDRS           # Target Headers files (for project based build systems)
     _SKETCHES       # Target sketch files
     _LIBS           # Libraries to linked against target
     _BOARD          # Board name (such as uno, mega2560, ...)
     _PORT           # Serial port, for upload and serial targets [OPTIONAL]
     _SERIAL         # Serial command for serial target           [OPTIONAL]
     _NO_AUTOLIBS    # Disables Arduino library detection (default On)


So to create a target (firmware image) called blink, composed of blink.h and blink.cpp source files for the Arduino Uno, you write the following:

    set(blink_SRCS  blink.cpp)
    set(blink_HDRS  blink.h)
    set(blink_BOARD uno)

    generate_arduino_firmware(blink)

To enable firmware upload functionality, you need to add the _PORT settings:

    set(blink_PORT /dev/ttyUSB0)

To enable serial terminal, add the _SERIAL setting (@INPUT_PORT@ will be replaced with the blink_PORT setting):

    set(blink_PORT picocom @INPUT_PORT@ -b 9600 -l)






3. Defining libraries
=====================

Creating libraries is very similar to defining a firmware image, except we use the generate_arduino_library command. The syntax of the settings is the same except we have a different list of settings:

     _SRCS           # Library Sources
     _HDRS           # Library Headers
     _LIBS           # Libraries to linked in
     _BOARD          # Board name (such as uno, mega2560, ...)
     _NO_AUTOLIBS    # Disables Arduino library detection

Lets define a simple library called blink_lib, with two sources files for the Arduino Uno:


    set(blink_lib_SRCS  blink_lib.cpp)
    set(blink_lib_HDRS  blink_lib.h)
    set(blink_lib_BOARD uno)

    generate_arduino_firmware(blink_lib)

Once that library is defined we can use it in our other firmware images... Lets add blink_lib to the blink firmware:

    set(blink_SRCS  blink.cpp)
    set(blink_HDRS  blink.h)
    set(blink_LIBS  blink_lib)
    set(blink_BOARD uno)

    generate_arduino_firmware(blink)
