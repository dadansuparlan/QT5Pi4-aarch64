# QT5Pi4-aarch64
How To Cross Compile Qt5 For Raspberrypi4 using aarch64-linux-gnu (works for Rpi4B and CM4)

## Configuration For Raspberrypi
- Download Image Raspberrypi Image : 2022-09-22-raspios-bullseye-arm64.img.xz
  - Link : https://downloads.raspberrypi.org/raspios_full_arm64/images/raspios_full_arm64-2022-09-07/2022-09-06-raspios-bullseye-arm64-full.img.xz
- Flash to Raspberrypi using Raspberrypi Imager
  - link : https://www.raspberrypi.com/software/
- on Raspberrypi
```
sudo nano /etc/apt/sources.lists
```
> remove # on all deb-src...... save and close nano text by "ctrl + x" and "y"
> 
![image](https://github.com/dadansuparlan/QT5Pi4-aarch64/assets/54036971/86fc5eb7-f19f-4caf-9d33-55b82c773a5b)
```
sudo apt-get update --allow -releaseinfo-change
```
```
sudo apt-get full-upgrade
```
```
sudo reboot
```
```
ldd --version
```
> check make sure version of glibc 2.31 or above

![image](https://github.com/dadansuparlan/QT5Pi4-aarch64/assets/54036971/e001ecf4-15a9-42be-b24b-df7cec5cf54f)

```
sudo apt-get build-dep qt5-qmake
```
```
sudo apt-get install libegl1-mesa libegl1-mesa-dev libgles2-mesa libgles2-mesa-dev wiringpi libnfc-bin libnfc-dev fonts-texgyre libts-dev libbluetooth-dev bluez-tools gstreamer1.0-plugins* libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libopenal-data libsndi07.0 libopenal1 libopenal-dev pulseaudio
```
```
sudo gpasswd -a pi render
sudo mkdir /usr/local/qt5pi
sudo chown pi:pi /usr/local/qt5pi
```

## Configuration Ubuntu Virtual Box on Host Computer
- download Virtual Box version 7.0 
  - link https://www.virtualbox.org/wiki/Download_Old_Builds_7_0
- add virtual Ubuntu 20.04.6-desktop-amd64.iso
  - link  http://repo.ugm.ac.id/iso/ubuntu/releases/focal/
    
- on virtualbox
```
sudo apt update
sudo apt upgrade
sudo apt install build-essential
sudo apt install -y gcc-aarch64-linux-gnu g++-aarch64-linux-gnu
sudo apt install -y libssl-dev
sudo apt install git wget rsync python
mkdir ~/raspi
cd ~/raspi
mkdir sysroot sysroot/usr
```
> now we will do syncronize lib and other dependences between ubuntu and raspberrypi
> find ip address raspberrypi, both must in the same network, for example in my case ip rasberrypi 192.168.1.6

```
rsync -avz pi@192.168.1.6:/lib sysroot
```
```
rsync -avz pi@192.168.1.6:/usr/include sysroot/usr
```
```
rsync -avz pi@192.168.1.6:/usr/lib sysroot/usr
```
```
wget https://raw.githubusercontent.com/riscv/riscv-poky/master/scripts/sysroot-relativelinks.py
```
```
chmod +x sysroot-relativelinks.py
```
```
./sysroot-relativelinks.py sysroot
```
```
git clone git://code.qt.io/qt/qt5.git qtsource
cd qtsource
git checkout v5.14.2
```
```
perl ini-repository --module-subset=qtbase,qtxmlpatterns,qtsvg,qtdeclarative,qtimageformats,qtgraphicaleffects,qtquickcontrols,qtquickcontrols2,qtvirtualkeyboard,qtwebsockets,qtwebglplugin,qtcharts,qtconnectivity,qtmultimedia,qtlocation,qtserialport,qtserialbus
```
> we will use mkspec device "linux-rasp-pi4-v3d-g++" this configuration of qmake.conf must be edit.
> 
> find file using file manager "qmake.conf" in folder "home/raspi/qtsource/qtbase/mkspecs/devices/linux-rasp-pi4-v3d-g++"
> 
> open qmake.conf using text editor
> 
> replace all text with :
```
include(../common/linux_device_pre.conf)
QT_QPA_DEFAULT_PLATFORM =

QMAKE_CFLAGS            = -march=armv8-a -mtune=cortex-a72
QMAKE_CXXFLAGS          = $$QMAKE_CFLAGS
DISTRO_OPTS            += deb-multi-arch
include(../common/linux_device_post.conf)

load(qt_config)
```

> save it and close text editor
> now we continue to build qt cross compile
```
cd
cd ~/raspi
mkdir qtpi-build
cd qtpi-build
```
```
../qtsource/configure -release -opengl es2 -device linux-rasp-pi4-v3d-g++ -device-option CROSS_COMPILE=aarch64-linux-gnu- -syroot ~/raspi/sysroot -opensource -confirm-license -make libs -prefix /usr/local/qt5pi -extprefix ~/raspi/qt5pi -hostprefix ~/raspi/qt5 -no-use-gold-linker -v
```
```
make -j4
make install
```
> now we copy all file Qt in folder qt5pi to raspberrypi using rsync

```
cd
cd ~/raspi
rsync -avz qt5pi pi@192.168.1.6:/usr/local
```
> We want to install qtmqtt or other modules but not find in source, so we must install manualy

```
cd
cd /raspi/qt5source
git clone git://code.qt.io/qt/qtmqtt.git -b 5.14.2
cd qtmqtt
~/raspi/qt5/bin/qmake
make
make install
cd
cd ~/raspi
rsync -avz qt5pi pi@192.168.1.11:/usr/local
```
>remove Folder and file not used for space drive

```
cd
cd ~/raspi
rm -rf qt5source
rm -rf qtpi-build
```
> now move to raspberrypi terminal or using ssh on ubuntu

```
pi@raspberrypi:~ $ echo /usr/local/qt5pi/lib | sudo tee /etc/ld.so.conf.d/qt5pi.conf
```
```
pi@raspberrypi:~ $ sudo ldconfig
```
- DONE STEP PREPARE CROSSCOMPILE TOOLS for Qt 5.14.2
- Now we can used Qtcreator for make App and build directly into raspberry pi

## Install Qt Creator and Configure Cross Compile
- on Host Computer ubuntu Virtual Box
- Download and install Qt opensource offline
  - link : https://download.qt.io/archive/qt/5.14/5.14.2/qt-opensource-linux-x64-5.14.2.run

```
cd
cd Downloads
wget https://download.qt.io/archive/qt/5.14/5.14.2/qt-opensource-linux-x64-5.14.2.run
chmod +x qt-opensource-linux-x64-5.14.2.run
./qt-opensource-linux-x64-5.14.2.run

```
>Install all module except android if not used
>after install done open Qt Creator
>Configure qt Creator
- First configure Device
>

![image](https://github.com/dadansuparlan/QT5Pi4-aarch64/assets/54036971/9671626b-7e17-42e1-80e8-6fac482f1db1)

![image](https://github.com/dadansuparlan/QT5Pi4-aarch64/assets/54036971/e5df9261-2494-4401-8196-204f8d33e30f)

> select Next -> Next -> Finish
> will show Failed, its ok select Close

 ![image](https://github.com/dadansuparlan/QT5Pi4-aarch64/assets/54036971/58e18394-4adc-47a5-b746-a347851138d0)

>fill Raspberry paswword and select Ok
>
![image](https://github.com/dadansuparlan/QT5Pi4-aarch64/assets/54036971/7916bfbd-320f-44d6-9246-033be0cffaae)

>will show successfully, select Close
- Second Configure Qt Version
> Select Kits -> Qt Version -> Add -> find file /home/raspi/qt5/bin/qmake
> Selct Open
>
![image](https://github.com/dadansuparlan/QT5Pi4-aarch64/assets/54036971/986d3179-2557-42be-bbe4-a63e42d62cb6)

>change Version Name : Qt %{Qt:Version} (qt5Raspi) and select Apply

![image](https://github.com/dadansuparlan/QT5Pi4-aarch64/assets/54036971/c2379d29-822d-4d7b-8047-31ac36b7d2ca)

- Third Configure Kits
  
>move to select Kits -> Add -> fill Name: Raspi4 -> Device type: Generic Linux Device -> Device: Raspi4 -> Sysroot : /home/raspi/sysroot ->  Compiler C: GCC(C, arm 64bit in /usr/bin C++: GCC(C++, arm 64bit in /usr/bin) -> Select Apply
>
![image](https://github.com/dadansuparlan/QT5Pi4-aarch64/assets/54036971/be1d8d1b-4193-4149-8453-eff52a74b714)

> select Raspi4 -> select Make Dafault -> select Apply adn Ok
>
![image](https://github.com/dadansuparlan/QT5Pi4-aarch64/assets/54036971/65336250-3432-4248-ac6b-d8f2b6c05801)

> All Finish for Qt Creator Configuration, Ready for Use
>
- Try Make Example App "Hello world"
> Create New Project QtQuick Application-Empty , Give Name, Next->Next->Next->select Raspi4->Next->finish
>  Change Text bellow #Default rules for deployment of Helloworld.pro with :
>
```
target.path = /home/pi/$${TARGET}/bin
!isEmpty(target.path): INSTALLS += target 
```
![image](https://github.com/dadansuparlan/QT5Pi4-aarch64/assets/54036971/540d28ae-5a27-4325-a5f1-080022cb9845)

> Select Project->Run->Enviroment Batch Edit->fill with Text :
```
DISPLAY=:0.0
QT_QPA_PLATFOMRTHEME=qt5ct
XAUTHORITY=/home/pi/.Xauthrity
XDG_SESSION_TYPE=x11
```
>select OK and Save Project

![image](https://github.com/dadansuparlan/QT5Pi4-aarch64/assets/54036971/d407ec34-83ee-466d-80b9-7fbc5cc4b16c)

> Helloworld.pro
![image](https://github.com/dadansuparlan/QT5Pi4-aarch64/assets/54036971/9c3547ca-20a2-4ca3-b37e-8f1e23384b8c)
>
> main.qml
>
> ![image](https://github.com/dadansuparlan/QT5Pi4-aarch64/assets/54036971/17bf1d6e-b5c6-4f43-abfd-9db72cc20f5d)
>
> Save Project and RUN
> will show on the Raspberry pi
>
> ![outputHelloworld](https://github.com/dadansuparlan/QT5Pi4-aarch64/assets/54036971/b35a93bd-d341-4e6a-b127-ac00e334a614)
>
- ALL DONE AND WORKS ENJOY










