
## FBTFT ile Beaglebone üzerinde Qt Uygulaması Geliştirilmesi ve TFT LCD Bağlantısının Kurulması


https://medium.com/@mustafaboyuk24/fbtft-ile-beaglebone-üzerinde-qt-uygulaması-geliştirilmesi-ve-tft-lcd-bağlantısının-kurulması-3e5cf506d90c

İlk olarak framebuffer cihaz  nedir bunun açıklamasını yapaılım.

* Framebuffer cihazı grafik donanımı için bir soyutlamanın olduğu yerdir. Bazı video donanımları framebuffer arabelleğini temsil eder ve uygulama yazılımının grafik donanımına iyi tanımlanmış bir arabirim yoluyla erişmesine izin verir, böylece yazılımın alt düzey arabirim öğeleri hakkında hiç bir şey bilmesine gerek kalmaz.

Elimizde bulunan lcd spi destekli olduğu için seri bağlantı kurularak tft-lcd' yi BBB' e kolaylıkla bağlayabileceğiz. Tabi buradaki önemli nokta lcd IC ' nin ki bu ili9341 oluyor, linux-kernel tarafından desteğinin bulunmasıdır.

Beaglebone içerisinde 2 adet lcd kontroller bulunmaktadır. Spi hattının device tree üzerinden aktif hale getirilmesi gerekmektedir.
Aşağıdaki resimde spi hattının bulunduğu pinler gösterilmiştir.
## SPI Pins

![cape-headers-spi](https://user-images.githubusercontent.com/47300390/210801497-a8b3de55-1ad1-4c9d-bc05-55894a2dc4f9.png)

LCD ile beagleboneblack arasındaki bağlantılar aşağıda tabloda belirtildiği gibi kurulmuştur.

```

   LCD			            BBB
   SDO (MISO)			P9_29 (spi1_d0)
   LED				P9_12 (GPIO1_28)
   SCK				P9_31 (spi1_sclk)
   SDI (MOSI)			P9_30 (spi1_d1)
   D/C				P9_16 (GPIO1_19)
   RESET			P9_15 (GPIO1_16)
   CS				P9_28 (spi1_cs0)
   GND				P9_01 (GND)
   VCC				P9_03 (3.3 V)
 
```


FBTFT driverını kullanabilmemiz için ilk olarak bu desteği eklememiz gerekmektedir. Bunu Linux-kernel içerisinde yapacağız.
`Device Drivers > Staging Drivers`  bölümüne giriyoruz, burada `Support for small TFT LCD display modules` kısmına giriyoruz ve karşımıza FB driver destekli LCD sürücülerin listesi çıkmaktadır.
Buradan ILI9341 LCD Controller desteği olan lcd yi seçiyoruz ve değişiklikleri kaydetip çıkıyoruz.

![Screenshot from 2023-01-11 08-48-35](https://user-images.githubusercontent.com/47300390/211740947-f64c1e7c-a14a-46e4-8e8e-1ab544ea4ffb.png)

Fb Destekli Cihazların listesi aşağıda gösterilmiştir.

![Screenshot from 2023-01-11 08-48-56](https://user-images.githubusercontent.com/47300390/211740975-10cc3be8-abd7-4247-9ce5-7bdd20985afa.png)

FBTFT sürücülerini aktive etmek lcdyi ayağa kaldırmak için yeterli olmayacaktır. Linux  hangi cihazların bağlı olduğunu bilmek isteyecektir, bu aşama ise device tree overlay ile gerçekleştirilecektir.
Device tree overlay (dto) bir merkezi device tree blob (dtb)dosyası üzerinde değişiklik (ekleme, aktive etme, pasif hale getirme) sağlar. DTO kullanan bir boatloader, çip üzerinde sistem (SoC) DT'yi koruyabilir ve ağaca düğümler ekleyerek ve mevcut ağaçtaki özelliklerde değişiklikler yaparak cihaza özel bir DT'yi dinamik olarak kaplayabilir.

Device tree overlay dosyasını ön yükleyici ile kullanabilmemiz için kernel içirisinde de bu desteği aktif hale getirmemiz gerekiyor. `Device Drivers > Device Tree and Open Firmware support` bölümüne girip Device Tree overlays kısmını işaretliyoruz.
![Screenshot from 2023-01-11 09-01-54](https://user-images.githubusercontent.com/47300390/211745939-1a50db25-a873-4a95-9b27-6f6c88c41a85.png)

Aşağıdaki overlay dosyasını oluşturuyoruz ve derliyoruz. Derleme sonunda .dtbo uzantılı bir dosya çıkmaktadır, bu dosya mevcut dtb dosyası üzerinde değişiklik yapıp spi hattını ve lcd cihazını aktif 
hale getirecektir. Lcd ile Beagleboneblack (BBB) pin bağlantıları kurulduğunda kernel mesajlarına bu bilgi resimde gösterildiği gibi yer alacaktır.

` dmesg | grep ili9341*  ` komutu ile kontol ettiğimizde ağaşıdaki çıktıları göreceksiniz.

![Screenshot from 2023-01-11 10-47-44](https://user-images.githubusercontent.com/47300390/211748079-2801cd72-0150-4db3-97e4-1f014574af98.png)
 FBTFT ve ILI9341 lcd sürücüsünü yüklü olup olmadığı arattımızda ` find / -name fb_ili9341* ve find / -name fbtft* ` aşağıdaki resimdeki çıktıyı görmekteyiz.
 
 ![Screenshot from 2023-01-11 10-53-28](https://user-images.githubusercontent.com/47300390/211749145-ab92e1cf-5552-4ea1-897b-be11144d45cc.png)

Buna göre fbtft.ko ve fb_ili9341.ko kernel modulümüz başarılı bir şekilde driverlarımız arasına eklenmiştir.
Beaglebone içerisinde yazdığımız overlay dosyasının uygulanıp uygulanmadığı konusunda şüphelerimiz varsa dosya sisteminde ` ls -l /sys/firmware/devicetree/base/chosen/overlays   `
komutu ile kontrol edebiliriz. 

![Screenshot from 2023-01-12 20-26-37](https://user-images.githubusercontent.com/47300390/212137514-4165b9f8-ff9c-4256-9fb6-6d8e5a4d044b.png)


```
Wed Jan 11 11:20:57 2023# ls -l /sys/firmware/devicetree/base/chosen/overlays
total 0
-r--r--r--    1 root     root            25 Jan  1  1970 BB-LCD_SPI_ILI9341
-r--r--r--    1 root     root             9 Jan  1  1970 name
```

Görüldüğü üzere BB-LCD_SPI_ILI9341 overlay dosyamız  devicetree klasöründe yer almaktadır. Bu bilgiyi overlay dosyansındaki ilk fragment yani fragment0 yardımıyla elde ettik.
Bu yüzden başlangıça bu fragment yapısını eklediğimizde uygulanan overlayleri device tree klasöründe görebileceğiz.


### Device Tree Compilation

Bu aşamada oluşturduğumuz BB-LCD_SPI_ILI9341.dts dosyasını derlememiz gerekmektedir. Bu işlemi device tree compiler yardımıyla yapabiliriz. 
Device Tree Compileri ubuntu için ` sudo apt-get install device-tree-compiler ` ile indirebiliriz.

* ` dtc -O dtb -o BB-LCD_SPI_ILI9341.dtbo BB-LCD_SPI_ILI9341.dts `   ile derliyoruz ve dtbo uzantılı overlay dosyasını elde ediyoruz.

Sonrasında `BB-LCD_SPI_ILI9341.dtbo` dosyasını root file system içerisinde `/lib/firmware` adlı bir klasör oluşturup içerisine kopyalıyoruz.

Oluşturduğumuz overlay dosyasında makrolar kütüphaneler header file dosyaları yer aldığından dolayı bu derleme işlemi giderek zorlaşabilir çünkü makro ve header dosyalarının yolunu compilera el ile vermek zorunda kalıyoruz bu yüzden alternatif olarak linux içerisinde yer alan overlay klasöründe ` touch arch/arm/boot/dts/overlays/BB-LCD_SPI_ILI9341.dts ` dosyamızı oluşturduk ve sonrasına 
` make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- dtbs ` komutu ile overlayimizi derlemiş olduk.
```
mustafa@mustafa:~/yedek_workspace/linux$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- dtbs
  DTCO    arch/arm/boot/dts/overlays/BB-LCD_SPI_ILI9341.dtbo
arch/arm/boot/dts/overlays/BB-LCD_SPI_ILI9341.dts:59.11-63.6: Warning (chosen_node_is_root): /fragment@0/__overlay__/chosen: chosen node must be at root node
```
Derleme sonucunda dtbo uzantılı dosyayı elde etmiş bulunmaktayız.


## SPI ILI9341 2.4 TFT LCD DEVICE TREE SOURCE FILE

Device tree kaynak dosyası TI tarafınan oluşturulmuştur. Biz bu dosyada statik olarak değişiklik yapmak yerine overlay ile dinamik olarak değişiklik yaptık.
fragment 1 'de  am33xx_pinmux device tree node 'ununa dair ayarlama mevcuttur. Burada spi ve lcd tarafından kullanılacak pinler ayarlanmıştır.
fragment 2 'de spi node 'una bir dair bir ayarlama mevcuttur. Fragment 2 eklemek istediğimiz cihazı göstermektedir, hedef cihaz spi1 olduğu için bu cihaz spi konrolcüye bağlı olacaktır.
fragment 2 'ye bağlı bir child node bulunmaktadır, tft lcd spi konrollurün bir alt node olmak zorundadır.


```

/*
 * Copyright (C) 2013 CircuitCo
 *
 * Virtual cape for SPI1 on connector pins P9.29 P9.31 P9.30 P9.28
 *
 * This program is free software; you can redistribute it and/or modify
 * it under the terms of the GNU General Public License version 2 as
 * published by the Free Software Foundation.
 */

/*
 *  LCD			    BBB
 *  SDO (MISO)                  N/C (spi1_d0)
 *  LED                         P9_12 (GPIO1_28)
 *  SCK			 P9_31 (spi1_sclk)
 *  SDI (MOSI)                  P9_30 (spi1_d1)
 *  D/C			 P9_16 (GPIO1_19)
 *  RESET		         P9_15 (GPIO1_16)
 *  CS			         P9_28 (spi1_cs0)
 *  GND                         P9_01 (GND)
 *  VCC			 P9_03 (3.3 V)
 */


#include <dt-bindings/gpio/gpio.h>
#include <dt-bindings/pinctrl/am33xx.h>

/dts-v1/;
/plugin/;

/ {

	compatible = "ti,beaglebone", "ti,beaglebone-black", "ti,beaglebone-green";

	/* identification */
	part-number = "BB-SPIDEV1";
	version = "00A0";

	/* state the resources this cape uses */
	exclusive-use =
		/* the pin header uses */
		"P9.31",	/* spi1_sclk */
		"P9.29",	/* spi1_d0 */
		"P9.30",	/* spi1_d1 */
		"P9.28",	/* spi1_cs0 */
		// "P9.42",	/* spi1_cs1 */
		/* the hardware ip uses */
		"spi1";
		
	
	/*
	 * Helper to show loaded overlays under: /proc/device-tree/chosen/overlays/
	 */
	
	fragment@0 {
		target-path="/";
		__overlay__ {

			chosen {
				overlays {
					BB-LCD_SPI_ILI9341 = __TIMESTAMP__;
				};
			};
		};
	};
	

	fragment@1 {
		target = <&am33xx_pinmux>;
		__overlay__ {
			/* default state has all gpios released and mode set to uart1 */
			bb_spi1_pins: pinmux_bb_spi1_pins {
				pinctrl-single,pins = <
					0x190 0x33	/* mcasp0_aclkx.spi1_sclk, INPUT_PULLUP | MODE3 */
					0x194 0x33	/* mcasp0_fsx.spi1_d0, INPUT_PULLUP | MODE3 */
					0x198 0x13	/* mcasp0_axr0.spi1_d1, OUTPUT_PULLUP | MODE3 */
					0x19c 0x13	/* mcasp0_ahclkr.spi1_cs0, OUTPUT_PULLUP | MODE3 */
					// 0x164 0x12	/* eCAP0_in_PWM0_out.spi1_cs1 OUTPUT_PULLUP | MODE2 */
				>;
			};
			
			lcd_ctrl_pinmux: lcd_ctrl_pins {
				pinctrl-single,pins = <
				0x040 0x17  /* gpio1_16  OUTPUT_PULLUP | MODE7 */
				0x078 0x17  /* gpio1_28  OUTPUT_PULLUP | MODE7 */
				0x04C 0x17  /* gpio1_19  INPUT_PULLUP | MODE7 */
				>;
			      };
		};
	};

	fragment@2 {
		target = <&spi1>;
		__overlay__ {
			#address-cells = <1>;
			#size-cells = <0>;

			status = "okay";
			pinctrl-names = "default";
			pinctrl-0 = <&bb_spi1_pins>;
			ti,pio-mode; /* disable dma when used as an overlay, dma gets stuck at 160 bits... */

		channel@0 {
			#address-cells = <1>;
			#size-cells = <0>;

			compatible = "rohm,dh2228fv";
			symlink = "bone/spi/1.0";

			reg = <0>;
			spi-max-frequency = <16000000>;
			spi-cpha;
		};

		channel@1 {
			#address-cells = <1>;
			#size-cells = <0>;

			compatible = "rohm,dh2228fv";
			symlink = "bone/spi/1.1";

			reg = <1>;
			spi-max-frequency = <16000000>;
		};

		lcd@0{

			compatible = "ilitek,ili9341";
			reg = <0>;
			bgr;
			fps = <30>;
			pinctrl-names = "default";
			pinctrl-0 = <&lcd_ctrl_pinmux>;
			spi-max-frequency = <32000000>;
	
			led-gpios = <&gpio1 28 0>;
			regwidth = <8>;
			buswidth = <8>;
			verbose = <3>;
			debug = <2>;
			
			rotate = <180>;
			reset-gpios = <&gpio1 16 1>;
			/*reset-gpios = <&gpio1 16 1>; */
			
			dc-gpios = <&gpio1 19 0>;
			
			status = "okay";
		};


		};
	};
};

```

Yukarıda verilen device tree overlay dosyasında fragment1'de spi haberleşmesinde kullanılacak olan pinlerin konfigurasyon işlemi gerçekleştirilmiştir.
Pin Mux işlemleri için detaylı bilgilere Beaglebone SRM dökümanından ulaşabiliriz. 
İkinci fragmentinde ise spi driveri ve lcd cihazı konfigure edilmiştir. Burada kritik kısım lcd cihazına verilecek olan parameterlerdir. 
Bu parametrelerin neye göre verildiğini linux/drivers/staging/fbtft/fbtft.h dosyasında belirtilmiştir.

Aşağıda ki kod parçasında lcd cihazının aldığı parametreler gösterilmiştir.
```
/**
 * struct fbtft_display - Describes the display properties
 * @width: Width of display in pixels
 * @height: Height of display in pixels
 * @regwidth: LCD Controller Register width in bits
 * @buswidth: Display interface bus width in bits
 * @backlight: Backlight type.
 * @fbtftops: FBTFT operations provided by driver or device (platform_data)
 * @bpp: Bits per pixel
 * @fps: Frames per second
 * @txbuflen: Size of transmit buffer
 * @init_sequence: Pointer to LCD initialization array
 * @gamma: String representation of Gamma curve(s)
 * @gamma_num: Number of Gamma curves
 * @gamma_len: Number of values per Gamma curve
 * @debug: Initial debug value
 *
 * This structure is not stored by FBTFT except for init_sequence.
 */
struct fbtft_display {
	unsigned int width;
	unsigned int height;
	unsigned int regwidth;
	unsigned int buswidth;
	unsigned int backlight;
	struct fbtft_ops fbtftops;
	unsigned int bpp;
	unsigned int fps;
	int txbuflen;
	const s16 *init_sequence;
	char *gamma;
	int gamma_num;
	int gamma_len;
	unsigned long debug;
};
```



## Qt Frameworkünün Çapraz Derleme için Hazırlanması

Qt yi beaglebone içerisinde kullanabilmemiz için gerekli paketleri buildroot yardımıyla indireceğiz.
ilk olarak `Target packages -> Graphic libraries and applications` bölümüne giriyoruz burada Qt5 modülünü ve kallanmak istediğimiz paketleri işaretliyoruz.

![Screenshot from 2023-01-11 09-08-01](https://user-images.githubusercontent.com/47300390/211779271-b697534b-7daa-4f15-a10c-7f31ddb0f9f6.png)

resim formatında bir ugulama geliştireceksek JPEG PNG GIF support seçilmelidir.
Grafik desteği için bir çok paket seçeneği mevcuttur. 
Burada mesa3d ve OSMesa OpenGl support için kullanılabilir. Biz bu çalışmada framebuffer driverini kullanacağımız için default graphical platform için `linuxfb` yi seçiyoruz.
Ayrıca alternatif font desteği için ` Target packages -> Fonts, cursors, icons, sounds and themes` içerisinde yer alan `DejaVu fonts` seçilebilr.

![Screenshot from 2023-01-11 09-06-53](https://user-images.githubusercontent.com/47300390/211779480-ffe52fe7-5834-4217-bec1-7de58263f001.png)


![Screenshot from 2023-01-11 09-06-46](https://user-images.githubusercontent.com/47300390/211779910-0209e5b7-1b1b-4dbe-9930-f6d25c92182f.png)


![Screenshot from 2023-01-11 09-05-45](https://user-images.githubusercontent.com/47300390/211780202-dd50f459-80f7-4d88-8277-4d5873c235c1.png)

Buldroot ile root file system oluştururken dikkat etmemiz gereken bir kaç durum vardır. Örneğin unuttuğumuz bir paket vs mevcut menuconfig içerisine tekrardan giriş yaptık ve ilgili paketi ekledik
sonrasında derlediğimizde bu paketin derlemeye tabi tutulduğuna log dosyasına bakarak bilgi alabiliriz.  `cat /home/mustafa/yedek_workspace/buildroot-2022.08.2/output/build/qt5base-2ffb7ad8a1079a0444b9c72affe3d19b089b60de/config.log` komutu ile hangi paketlerin derlendiği ilk kısımda gösterilmiştir.

```

mustafa@mustafa:~/yedek_workspace/buildroot-2022.08.2/output/build/qt5base-2ffb7ad8a1079a0444b9c72affe3d19b089b60de$ cat config.log 
Command line: -v -prefix /usr -hostprefix /home/mustafa/yedek_workspace/buildroot-2022.08.2/output/host -headerdir /usr/include/qt5 -sysroot /home/mustafa/yedek_workspace/buildroot-2022.08.2/output/host/arm-buildroot-linux-gnueabihf/sysroot -plugindir /usr/lib/qt/plugins -examplesdir /usr/lib/qt/examples -no-rpath -nomake tests -device buildroot -device-option CROSS_COMPILE=/home/mustafa/yedek_workspace/buildroot-2022.08.2/output/host/bin/arm-linux-gnueabihf- -device-option 'BR_COMPILER_CFLAGS=-D_LARGEFILE_SOURCE -D_LARGEFILE64_SOURCE -D_FILE_OFFSET_BITS=64  -Os -g0 -D_FORTIFY_SOURCE=1' -device-option 'BR_COMPILER_CXXFLAGS=-D_LARGEFILE_SOURCE -D_LARGEFILE64_SOURCE -D_FILE_OFFSET_BITS=64  -Os -g0 -D_FORTIFY_SOURCE=1' -optimized-qmake -no-iconv -system-zlib -system-pcre -no-pch -shared -no-feature-relocatable -no-optimize-debug -no-sse2 -kms -no-gbm -release -opensource -confirm-license -no-cups -no-zstd -no-sql-mysql -no-sql-psql -no-sql-sqlite -gui -system-freetype -no-harfbuzz -widgets --enable-linuxfb -no-directfb -no-xcb -no-opengl -qpa linuxfb -no-eglfs -openssl -fontconfig -no-gif -no-libjpeg -system-libpng -no-dbus -no-tslib -no-glib -no-icu -nomake examples -no-libinput -no-gtk -no-journald -no-syslog

```


Qt de oluşturduğumuz widget uygulamasını çapraz derleyip beaglebone içerisine atıyoruz 

Qt widget uygulamasını çalıştırdığımızda aşağıdaki çıktıyı göreceğiz.

![Screenshot from 2023-01-11 13-19-33](https://user-images.githubusercontent.com/47300390/211783042-0b999318-a1aa-4c76-8381-3926be8d130c.png)



![BBB_LCD](https://user-images.githubusercontent.com/47300390/211782070-b3c8e580-3923-47a4-86d3-a72f90fefdef.jpeg)

Eğer default graphical platform bülümünü boş bıraksaydık veya farklı bir grafik kütüphanesi mevcutsa ` <qt_program> -platform linuxfb:fb=/dev/fb0 `
ile beaglebone üzerinde programımızı çalıştırabilmek için ekstra parametre ile kullanacağımız platformu belirtmemiz gerekebilir.




## Kaynaklar 

https://github.com/notro/fbtft/wiki/Framebuffer-use



