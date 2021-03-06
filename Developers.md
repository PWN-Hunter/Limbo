## Limbo Emulator (QEMU) for Android

================================================================================

1. What is Limbo?

Limbo is a QEMU-based emulator for Android. It currently supports PC emulation 
for Intel x86 architecture.
For more information, instructions, guides, known issues, and downloads visit:
https://github.com/limboemu/limbo

===============================================================================

2. Requirements:

    Android SDK
    Android NDK r14b/gcc (clang is optional but generates slower runtime binaries)
    Android Studio (4.1 prefered)
    Android device with Android OS 9.0 (Pie) and above
    Linux Desktop pc (Ubuntu prefered)
    Make sure you have the following packages installed, if not run:
    sudo apt install make autoconf automake git python binutils libtool-bin pkg-config flex bison

===============================================================================

3. Known Issues:
    https://github.com/limboemu/limbo/issues

===============================================================================
4. Setup Environment

    a. Update variables for paths to NDK directories for your Build Environment
     in ./limbo-android-lib/src/main/jni/android-limbo-config.mak
     
    b. Configure your PATH variable add the NDK Directory so makefile can find ndk-build
    Examples:
    For bash:
    export PATH=$PATH:/home/dev/tools/ndk/android-ndk-r14b

5. Get and Patch libraries

    #Make sure you're under the jni directory
    cd ./limbo-android-lib/src/main/jni

    #Note: if some of these file links don't download with wget use your browser to download them

    ##### Get QEMU (modify the path to .../qemu-4.0.0.tar.xz if you compile with QEMU 4.0.0 instead)
    wget http://download.qemu-project.org/qemu-2.9.1.tar.xz -P /tmp/
    tar -xJf /tmp/qemu-2.9.1.tar.xz
    mv qemu-2.9.1 qemu

    ##### GET glib
    wget https://ftp.gnome.org/pub/GNOME/sources/glib/2.56/glib-2.56.1.tar.xz -P /tmp/
    tar -xJf /tmp/glib-2.56.1.tar.xz
    mv glib-2.56.1 glib

    ##### GET libffi
    wget https://sourceware.org/pub/libffi/libffi-3.2.1.tar.gz -P /tmp/
    tar -xzf /tmp/libffi-3.2.1.tar.gz
    mv libffi-3.2.1 libffi

    ##### GET pixman
    wget https://www.cairographics.org/releases/pixman-0.34.0.tar.gz -P /tmp/
    tar -xzf /tmp/pixman-0.34.0.tar.gz
    mv pixman-0.34.0 pixman

    ##### GET SDL2
    wget https://www.libsdl.org/release/SDL2-2.0.8.tar.gz -P /tmp/
    tar -xzf /tmp/SDL2-2.0.8.tar.gz
    mv SDL2-2.0.8 SDL2

    Now you should have this directory structure:
    jni/
        android-config/
        compat/
        glib/
        libffi/
        limbo/
        patches/
        pixman/
        qemu/
        SDL2/


    ### If you want to compile with QEMU 2.9.1:
    cd ./limbo-android-lib/src/main/jni/qemu/
    patch -p1 < ../patches/qemu-2.9.1.patch

    ### Or if you want to compile with QEMU 4.0.0:
    cd ./limbo-android-lib/src/main/jni/qemu/
    patch -p1 < ../patches/qemu-4.0.0.patch
    For QEMU 4.0.0 you also need to modify file android-config/android-qemu-config.mak:
        set to false: USE_QEMUSTAB ?= false
        set to true: USE_SLIRP_LIB ?= true
        comment line: #PIXMAN = --with-system-pixman
        #MISC += --disable-capstone
        #MISC += --disable-malloc-trim
        set to false: USE_SDL_ABI ?= false
     Also if you want to enable to explicitly enable MTTCG modify Config.java:
        set to true: public static boolean enableMTTCG = true;

    ### Apply glib patch for Limbo:
    cd ./limbo-android-lib/src/main/jni/glib/
    patch -p1 < ../patches/glib-2.56.1.patch


    ### Other QEMU versions:
    If you want to distribute Limbo build with other QEMU versions, create your own patch like this:
    cd /limbo-android-lib/src/main/jni/qemu/
    diff -ru --no-dereference /tmp/qemu-x.x.x . | grep -v '^Only in' > ../patches/qemu-x.x.x.patch

===============================================================================
5. Build

    a. To build the NDK part of the app make sure you're under the jni directory:
        cd limbo-android-lib/src/main/jni

    Make sure you update the android-config/android-limbo-config.mak file with the NDK path you have installed:
    NDK_ROOT = /home/dev/tools/ndk/android-ndk-r14b

    b. From Android Studio import BOTH the Android library limbo-android-lib AND the module for the guest
        architecture you need (x86,arm,ppc,sparc) ie limbo-android-x86.
        
    c. Build the native libraries:
        cd limbo-android-lib/src/main/jni:

        To build Limbo Emulator:
            export BUILD_HOST=<EABI>
            export BUILD_GUEST=<GUEST_ARCH>
            export NDK_DEBUG=<ENABLE_DEBUG>
            make limbo

        where:
            EABI is the Android device type (host arch): armeabi-v7a, arm64-v8a, x86, x86_64
            GUEST_ARCH is the Emulator type: x86_64-softmmu,aarch64-softmmu,sparc64-softmmu, ppc64-softmmu
            ENABLE_DEBUG is 1 (optional)

        If you want to remove ALL previously compiled native objects and libraries:
        make clean

        If you're building apk for multiple host architectures you need to do in betweeen builds:
        make distclean

        If you're building apk for multiple guest architectures you can specify them with commas:
        BUILD_GUEST=x86_64-softmmu,aarch64-softmmu

        Examples:
        1) To build Limbo x86 Emulator for ARM phones type:
            export BUILD_HOST=armeabi-v7a
            export BUILD_GUEST=x86_64-softmmu
            make limbo

        2) To build Limbo x86 Emulator for ARM64 phones type:
            export BUILD_HOST=arm64-v8a
            export BUILD_GUEST=x86_64-softmmu
            make limbo

        3) To build Limbo ARM Emulator for ARM64 phones type:
            export BUILD_HOST=arm64-v8a
            export BUILD_GUEST=aarch64-softmmu
            make limbo

        4) To build Limbo x86 Emulator for Intel x86 32bit phones/tablets/PCs type:
            export BUILD_HOST=x86
            export BUILD_GUEST=x86_64-softmmu
            make limbo

        5) To build Limbo x86 Emulator for Intel x86 64bit PCs type:
            export BUILD_HOST=x86_64
            export BUILD_GUEST=x86_64-softmmu
            make limbo

        6) To build Limbo ARM Emulator for ARM64 phones for debugging type:
            export NDK_DEBUG=1
            export BUILD_HOST=arm64-v8a
            export BUILD_GUEST=aarch64-softmmu
            make limbo

        7) To build multiple Limbo emulators for ARM phones type:
            export NDK_DEBUG=1
            export BUILD_HOST=arm64-v8a
            export BUILD_GUEST=x86_64-softmmu,aarch64-softmmu
            make limbo

       You should now have the following libraries in these 2 folders:

       limbo-android-lib/src/main/jniLibs/<EABI>/
         libcompat-iconv.so
         libcompat-intl.so
         libcompat-limbo.so
         libcompat-SDL2-ext.so
         libglib-2.0.so
         liblimbo.so
         libpixman-1.so
         libSDL2.so

       limbo-android-<GUEST_ARCH>/src/main/jniLibs/<EABI>/
         libqemu-system-xxx.so

        Note:
            When you build the apk in Android Studio it will contain the libraries from both
            folders so you don't need to copy files manually.

    d. Build the Android apk for the corresponding guest using Android Studio.
        Make sure the *.so libraries are zipped in the final .apk

    e. If you want to build the debugging version:
           Set variables in Config.java:
             debug = true;
           Modify android-config/android-limbo-config.mak and point to a configuration
         with no optimization:
         USE_OPTIMIZATION ?= false

        Follow the steps to build the native libraries as described above
        export NDK_DEBUG=1

        Important:
           From Android studio click Build> Rebuild Project and Run > Debug
         
    f. To start debugging the native code for a particular guest arch type
        for x86 guest:
        make ndk-gdb PKG_NAME=com.limbo.emu.main

        for arm guest:
        make ndk-gdb PKG_NAME=com.limbo.emu.main.arm

        for sparc guest:
        make ndk-gdb PKG_NAME=com.limbo.emu.main.sparc

        for ppc guest:
        make ndk-gdb PKG_NAME=com.limbo.emu.main.ppc

        Note: you might want to disable signal handling in gdb:
        handle all nostop noprint

        To catch a SIGSEGV:
        handle 11 stop print

        Useful commands:
        To set breakpoints:       b <file.c>:<code_line>
        To step statement:        s
        To go to next statement:  n
        To continue:              c
        To halt(interrupt):       ctrl-c
        To see stack (backtrace): bt

===============================================================================
6. Development

    a. Codes Changes for Android compatibility are in patch files marked with __ANDROID__

    b. Similarly for LIMBO functionality code changes are tagged with __LIMBO__

    c. Important Configuration files (You'll need to update this with the NDK path before compiling):
        limbo-android-lib/src/main/jni/android-config/android-limbo-config.mak

    d. Advanced QEMU config files:
        limbo-android-lib/src/main/jni/android-config/android-qemu-config.mak

    d. Advanced Device Configuration files:
        limbo-android-lib/src/main/jni/android-config/android-device-config/*.mak

    e. Important Makefiles:
        limbo-android-lib/src/main/jni/Makefile
        limbo-android-lib/src/main/jni/Android.mk
        limbo-android-lib/src/main/jni/Application.mk
        limbo-android-lib/src/main/jni/android-limbo-build.mak
        limbo-android-lib/src/main/jni/android-qemu-build.mak

    f. Frontend UI options configuration see: Config.java
        
===============================================================================
7. Run

    a. Installing a full Qwerty keyboard for Android like Hacker's keyboard 
        from the Google Android store. Make sure you use Transparent theme 
        and Direct Draw found under Theme settings.
    b. Start the Limbo app and choose CPU, Memory (~8-64MB),etc..
    c. Choose a bootable disk image(s) for CDRom, Floppy, and a HDD image
    d. Start the virtual machine.
    e. For more instructions and guides visit: 
        https://github.com/limboemu/limbo
    f. Have fun!

===============================================================================
8. Changelog
See limbo-android-lib/src/main/assets/CHANGELOG for release notes

===============================================================================
9. License

Limbo PC Emulator is released under GPL v2 License.
All icons unders /res are from Gnome Project (GPL v2 License)
See file COPYING under root directory
and LICENSE under limbo-android-lib/src/main/assets

Other source included are released under their own license please view Licenses under each subdirectory

===============================================================================

Endofdoc
