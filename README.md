# mtk-gps-debugging

__The missing MTK driver GPS debugging information, translated from http://bbs.raindi.net/thread-24797-1-1.html. I am not the author of this, just storing a translated version here for easier reference. Please feel free to send pull requests to improve this documentation.__

**Basic knowledge of** **GPS** 

**Ø 1.1 ) What is the difference between GPS positioning and network positioning?**

GPS positioning requires the participation of satellites, and the device uses the searched satellite signals to calculate the location of the device.

Network positioning refers to the use of base stations and WIFI MAC to obtain a rough location.

**Ø 1.2 ) What is the difference between 3D positioning and 2D positioning?**

3D generally refers to positioning completed using at least 4 satellites.

2D generally uses 3 satellites to complete the positioning process.

Compared with 2D positioning, 3D positioning has higher positioning accuracy.

**Ø 1.3 ) Does the satellite distribution have a great influence on GPS performance ?**

Yes. GPGGA and GPGSA have this data in NMEA. The smaller the value, the better, and it is recommended to be less than 2.

The device is located at a certain point on the earth, which can be considered to be above the earth, with 4 quadrants, with an elevation angle of 0 to 90 degrees, as shown in the figure below.

A good satellite distribution must meet the following conditions:

-   Satellites in each quadrant;
-   Satellites cannot be at the same elevation angle.
-   Satellites with low elevation angles are more likely to be jammed and have longer transmission distances, which are prone to problems.  
    

  

Explain the common satellite distribution:

-   In the open sky environment, there are satellites in each quadrant.
-   In the environment by the window, you can only see the satellites in half of the sky, that is, there are satellites in the semicircle.
-   In the busy city, on the streets surrounded by tall buildings, only satellites with high elevation angles can be seen.  
    

  

Due to the poor satellite distribution, it will affect the positioning time and positioning accuracy, and cause users to experience poor positioning performance.

**Ø 1.4 ) How to synchronize GPS time to local time?**

Step1: Set the time option in the settings to synchronize with GPS time.

Step2: Turn on the location service in the settings.

Step3: Open any map application, after the positioning is successful, you can see the time synchronization. For example, use YGPS in engineering mode to locate APK.

**Ø 1.5 ) What is the satellite number of each satellite system?**  
![](http://bbs.raindi.net/data/attachment/forum/201908/17/135902ebuazabzx2y93n2g.jpg)  
**Ø 1.6 ) What is the accuracy of GPS positioning?**

The positioning accuracy is strongly related to the test environment and the hardware performance of the equipment.

The data that can be given: open sky environment, the signal strength of 6 satellites is greater than 40db, CEP67=3 meters.

  
**Special knowledge of** **MTK ALPS GPS** 

**Ø 2.1 ) What does FULL start , COLD start , WARM start and HOT start mean?**

The most important auxiliary information in the positioning process includes time, location, and ephemeris.

FULL start: There is no auxiliary information. It is equivalent to the scenario where the end user uses the positioning application after buying the mobile phone for the first time.

COLD start: There is time for auxiliary information, the end user will not encounter this scene.

WARM start: There is time and location assistance information. The end user's current positioning is more than 2 to 4 hours away from the last positioning.

HOT start: With all the auxiliary information, the end user's positioning is less than 2 to 4 hours from the last positioning.

So the scene that end users often encounter is WARM/HOT start.

**Ø 2.2 ) What is the TTFF of various startup methods ?**

The results of TTFF are strongly related to the test environment, test methods, and the GPS performance of the hardware.

The data given by MTK is based on the open sky environment, there are 6 satellites SNR "40db.

FULL start TTFF: less than 50s.

COLD start TTFF: less than 40s.

WARM start TTFF: less than 35s.

HOT start TTFF: less than 5s.

**Ø 2.3 ) What are the assisted positioning technologies?**  
![](http://bbs.raindi.net/data/attachment/forum/201908/17/135415f47a4kou4ztvptrv.jpg)  
  
These three technologies can be turned on at the same time, and when auxiliary information is provided at the same time, MTK's GPS algorithm will accept them and will not conflict.  
**Ø 2.4 ) How to get MNL Version quickly ?**

MNL is the abbreviation of Mediatek Navigation Library, and the version number is marked by date, for example MNL_VER_14051401ALPS05_3.60_09.

Turn on the location service in the settings, please enter YGPS-----"INFORMATION--àMNL version in the engineering mode to see the version number.

**Ø 2.5) Does Galieo support?**

The software does not support it.

**Ø 2.6) How often does Gps report data during its work?**

1HZ.

The highest can be adjusted to 5HZ, but it is not recommended to modify to 5HZ, because it will bring high power consumption.

Please set g_is_1Hz=5 in the linux_gps_init interface. For specific modification methods, please refer to the following:

Modify init.rc:

service mnld /system/xbin/mnld

class main

group nvram gps inet misc sdcard_rw sdcard_r media_rw

socket mnld stream 660 gps system.

**Ø 2.7) Does it support SBAS ?**

QZSS/WASS/EGNOS/MSAS/GAGAN.

**Ø 2.8) What is the support for GNSS ?**

GNSS refers to multi-satellite systems, such as GPS, GLONASS, BEIDOU.

  

![](http://bbs.raindi.net/data/attachment/forum/201908/17/140100q30xtnbbgc9cg4xa.jpg)

  
**Ø 2.9) Is there a sleep mechanism during GPS operation?**

No.

**Ø 2.10) The frequency deviation of the TCXO materials used by the GPS chip is different. Does the software need to be configured?**

As long as the GPS chip is not 6620, the software does not need to be configured.

**Ø 2.11) How does the software configure which satellite system to use?**

method one:

Find the mnl.prop file (path: /data/misc/, if it does not exist, please create the file)

Open the file, add GNSS_MODE=value and save it in push to /data/misc/.

Method Two

Modify the value of GNSSOPMode in the mnl_config variable in mnl_process_6620.c.

For the value range of 3332:

For the value range other than 3332:

For 6625L, the default is GPS+GLONASS, you can switch to GPS+BEIDOU, but you cannot support GPS+GLONASS+BEIDOU at the same time;

For MT3332, the default is GPS+GLONASS, you can switch to GPS+BEIDOU, but you cannot support GPS+GLONASS+BEIDOU at the same time;

For MT6630, the default is GPS+GLONASS+BEIDOU;

pay attention:

If you modify the GNSS configuration while the system is running, please delete /data/misc/mtkgps.dat in order for the modified configuration to take effect.

If GPS is running, remember to turn off GPS after modifying the configuration, and then delete /data/misc/mtkgps.dat.

LOG related

**Ø 3.1) Why do MTK engineers always need to provide gps debug log ?**

Because the GPS debug log will contain detailed information during the positioning process, such as whether a satellite ephemeris has been resolved, whether the satellite has been involved in positioning, whether there is interference, whether the clock is stable, and so on.

With this log, MTK engineers can analyze the GPS problem thoroughly, which is of great help in finding the root cause of the problem.

**Ø 3.2) Where is the Gps debug log stored?**

Stored in /data/misc/, its name is gpsdebug.log. time, for example, gpsdebug.log.20141124155650.

Therefore, to retrieve the log, root privileges are required.

**Ø 3.3) Can the storage directory of Gps debug log be modified?**

Can.

Please modify as follows:

#define LOG_FILE “/data/misc/gpsdebug.log”

**Ø 3.4) What log can be captured to facilitate MTK engineers to quickly analyze GPS problems.**

GPS issues submitted by customers are divided into the following 3 categories;

A, GPS cannot work, that is, GPS is not activated at all. Need mtklog (APlog, Modem log, Netlog).

To determine this is the problem, you need to follow the steps below to confirm.

Step1: Turn on the location service in the setting menu.

Step2: check engineering mode-----"Location---"YGPS----->INFORMATION--àMNL version

For UNKNOWN. If it is UNKNOWN, it means that the GPS has not been activated at all.

B, GPS performance is poor. Need mtklog (APlog, Modem log, Netlog), gps debug log.

How to open GPS debug log?

Please enter the engineering mode before starting the test-----"Location---"YGPS----->NMEA LOG--à find the dbg2file button, when the display becomes Disable dbg2file[Need Restart], it means the setting is successful.

Then please log out of YGPS.

Now you can start your test.

C, AGPS certification test failed. Need mtklog APlog, Modem log, Netlog), gps debug log, instrument log.

**Ø 3.5) Can GPS NMEA sentences appear in mtklog ?**

Can.

Engineering mode -----"Location---"YGPS----->NMEA LOG--à find the dbg2ddms button, the default display becomes Disable dbg2ddms[Need Restart], which means it has been opened, so that NMEA sentence can be used The log is in mtklog—"mobile log---"main log.

**Ø 3.6) How to judge the GPS software is working normally through log ?**

Please check whether NMEA Sentence appears in the log. If it appears, it means the software is normal.

**Ø 3.7) How to determine that the navigation system selected is the software configuration**

Check the value of GNSSOPMode from the log.

**Code Flow related** **Ø 4.1) Download process of** **EPO** **.**  
**Ø 4.2) Develop GPS tool , refer to the process of CWtest , meta gps , ftm gps .**

**Ø 4.3) GPS LNA GPIO control process.**  
**GPS cannot find satellites** **Ø 4.1) Confirm that the GPS software is working properly. There are satellite signals in the open sky environment, but why can’t my device find a satellite? How should I check for such problems?**

The current GPS hardware design generally requires a processing circuit before the satellite signal enters the chip as follows:

From the software point of view, you need to check whether LNA is enabled, that is, whether ANT_SEL0 is pulled high. If you find that it is not pulled up, you need to check whether there is a problem with the dws file configuration. Please refer to the figure below, especially the red part must be correct.

**Ø 4.2) For 6752/6732 platform for GPS LNA GPIO of pin invalid control, resulting in not deal with the problem of how Star Search?**

A. If the MT6306 GPIO port is not used as GPS LNA PIN on the hardware, please modify the project makefile,

Set MTK_MT6306_SUPPORT to no.

For Android 5.0, the modifications are as follows:

1. device/oppo/oppo6752_lwt_cu/ProjectConfig.mk 
```
MTK_MT6306_SUPPORT = no;
```
2. kernel-3.10\drivers\misc\mediatek\Kconfig.drivers

```
#config MTK_MT6306_SUPPORT

# tristate "MediaTek MT6306 GPIO Controller support"

# default y

3. \kernel-3.10\arch\arm64\configs\oppo6752_lwt_l_debug_defconfig

#CONFIG_MTK_MT6306_SUPPORT = y;
```

Remember and refer to FAQ10165 to configure the GPS LNA PIN correctly.

B. If the MT6306 GPIO port is used as GPS LNA PIN on the hardware, please use GPIO7 according to the reference design.

If you are using other GPIO of MT6306, please modify the following code
```
wmt_plat_alps.c

#ifdef MTK_MT6306_SUPPORT

... ...

#define GPIO_GPS_LNA_PIN GPIO7

#endif
```
Change to
```
#ifdef MTK_MT6306_SUPPORT

... ...

#define GPIO_GPS_LNA_PIN GPIOX

#endif
```
GPIOX is the GPIO port specifically used by your company.

C. If the problem is still not solved after the modification, it is generally because the modification did not take effect. You can check whether the modification is correct by means of check log. If MT6306 is used, "wmt_plat_gps_lna_ctrl" will be printed.

  
**Coclock related** **Ø 5.1) What platforms currently support Coclock ?**

6572, 6582, and 6592 are all supported. These platforms are generally matched with 6625 and 6627.

**Ø 5.2) What is the Coclock solution?**

Coclock will save a TCXO material, which is the material in the red box in the figure below. So the source of the clock source will be obtained from another place, currently from MT6166.

**Ø 5.3) Why does Coclock need to be calibrated?**

At present, GPS/WIFI/FM/BT all require a 26M clock, especially GPS, which requires a relatively high clock. If it is not calibrated, GPS will encounter various unexpected behaviors.

The main purpose of calibration is to calibrate a frequency versus temperature curve, so that the software can use the curve to learn the accurate value of the clock.

The current production line is calibrated, and the temperature range for calibration is relatively limited. The curves in the rest of the temperature range are calculated.

**Ø 5.4) What is the difference between the Coclock solution and the TCXO solution in end user usage?**

TCXO will provide a more accurate clock.

The clock provided by Cocock is relatively inaccurate. It needs software compensation, and it takes a long time to learn before it can achieve the same effect as TCXO.

In the case of Coclock, every time the positioning process is performed at a different temperature, the frequency temperature curve will be learned.

When the frequency vs. temperature curve, the wider the temperature range, the more complete the learning, and the better the end user will experience.

**Ø 5.5) How to configure the software under Coclock ?**

For 6572/6582, please refer to the following:

Please refer to MT6572_6582_GPS_clock_load_setting_SOP_v1.2.pdf.

For 6571/6592/6580, please refer to the following:

Please refer to MT6571_6592_GPS_clock_load_setting_SOP_v1.1.pptx.

**Ø 5.6) Where does the log indicate the coclock solution I chose ?**

In Kernel log, Co_clock_flag=1 means gps co clock; 0 means TCXO.

**Ø 5.7) How to judge that the calibration is successful?**

Check whether the values ​​of CO and C1 in the check log are 0. A value of 0 indicates that the calibration has failed, otherwise the calibration is successful.

mnl_linux: linux_gps_init: init_cfg.C0 = 0

mnl_linux: linux_gps_init: init_cfg.C1 = 0

**Ø 5.8) How to check the stability of clock ?**

The stability of the clock has a great influence on the performance of the gps, so when designing, we must strive to have a stable clock.

The current clock indicator data developed by MTK can be referred to as follows:

If there is no thermal interference, it is recommended that the clock drift is less than 2.5ppb/s.

In the case of thermal interference, it is recommended that the clock drift is less than 10ppb/s.

To check the clock of the product, you need to grab a gps debug log that has been positioned for 10 minutes, and import the log into the gps doctor tool to see the clock drift data.

  
**AGPS certification related** **Ø 6.1) Why do I need a 3D fix before testing AGPS sensitivity ?**

Before the AGPS certification test, the tester is usually required to complete a 3D fix of the device and keep the positioning for 1 minute.

This is because after the positioning is successful, the GPS algorithm has a more accurate understanding of the clock. This is more guarantee for the subsequent PASS certification test.

**Ø 6.2) Before the AGPS certification test, what are the software and hardware inspections required?**

If you plan to pass the AGPS certification test for this item, please inform MTK of the item in advance so that MTK can do the preliminary inspection.

Hardware check:

Antenna efficiency recommendations;

Conductivity:

Software check:

GPS driver patch check

Gps hal patch check

Agps dameon patch check

Patch inspection of AGPS modem

**Ø 6.3) What are the general locations for AGPS certification testing?**

TMO, AT&T, CMCC, Sporton laboratories.

**Ø 6.4) Is there an SOP for certification testing ?**

Please download Smart_Phone_AGPS_Performance_Test_SOP_for_Customer.pdf from DMS.

  

Test related

**Ø 7.1) How to test the TTFF of FULL start , WARM start , COLD start and HOT start ?**

Please use YGPS in engineering mode to test with FULL, COLD, WARM, HOT buttons.

**Ø 7.2) How to perform GPS field trial test?**

Please refer to MTK_GPS_Phone_Field_Test_20111021.pdf (AGPS, EP0 test please refer to this document).

**Ø 7.3) How to test the auxiliary effect of EPO ?**

At present, MTK's solution has integrated EPO and Hotstill into the system by default, and they are all turned on by default. There is no menu for the tester to close the EPO.

Please follow the steps below:

Step1: Make sure that the network is OK.

Step2: Use YGPS to set the Hotstill button in INFORMATION to display it as Enable Hotstill[Need Restart].

Step3: Exit YGPS, turn on YGPS again, and complete the 3D fix, and keep the positioning state for 1 min.

Step4: Turn off the network.

Step5: Use adb to delete /data/misc/BEE.bin and /data/misc/ARC.bin.

Step6: In YGPS, execute WARM start to see if the positioning can be completed within 5s, please test under open sky.

**Ø 7.4) How to test the auxiliary effect of Hotstill ?**

At present, MTK's solution has integrated EPO and Hotstill into the system by default, and they are all turned on by default.

Please follow the steps below:

Step1: Make sure the network is closed, use adb to delete /data/misc/EPO*

Step2: Use YGPS to set the Hotstill button in INFORMATION to display it as Disable Hotstill[Need Restart].

Step3: Exit YGPS, then open YGPS, and complete the 3D fix, and keep it for 10 minutes.

Step4: In YGPS, execute WARM start to see if the positioning can be completed within 5s, please test under open sky.

**Ø 7.6) The customer has developed its own test methods and standards, and after the test fails , why ?**

For this kind of problem, it is more difficult to sort out. Because as long as you test, there are many determinants of success or failure. For example, the GPS hardware performance of the mobile phone itself, whether the software patch is available, what kind of test method is and so on.

It is recommended to sort it out as follows:

-   Test whether the phone has elapsed HW Quality test check too ? Conductive CNR? Open sky CNR? Is there a de-sense question ? Is there a 2D Pattern or OTA pattern?
-   Does the phone have Clock and other issues ?
-   Phone HW version ? Phone SW version ? MNL version =? Whether Patch are on ?
-   Can debug log be obtained from GPS log?
-   If AGPS is involved , is there a mobile log?
-   Which phone is Benchmark ? Test results ? Comparison of CNR between MTK and Benchmark in the same environment ?
-   What is the test method ? ( Is the Cold Start of YGPS ? Or GPS test of Clear AGPS? How is the hot-start AGPS measured ?)
-   What is the test environment ? Is there a map of the surrounding environment ?  
    

So when this kind of problem occurs, please clarify these problems first. If everything is clarified, and there are still problems, please discuss these clarified problems with MTK to find the root cause.

GPS data

**Ø 8.1 ) Please find the following from DCC :**

MTK_GPS_Phone_SQC_Test_20120220_Sim.pdf (please refer to this document for GPS Field try)

MTK_GPS_Phone_Field_Test_20111021.pdf (Please refer to this document for AGPS, EP0 test)

HotStill_Standard_Testing_Android.pdf (Please refer to this document for Hotstill test)

YGPS_User_Manual.pdf

Android GPS Customer Document-MT6620.pdf

MT6628_GPS_Training_Material.pptx

GPS-Logs-SOP.pptx

**Ø 8.2 ) What GPS resources does eCourse on MTK online have ?**

GPS introduction

GPS Software Flow

GPS log analyze

MAUI GPS porting and debug

GPS training_GPS Tools and Test Procedule

GPS training_MTK GPS solution

GPS training_MTK GPS spec

**Ø 8.3 ) What information does porting MT3332 have?**

At present, MT3332 is not supported by default on 6589, 6572, 6582, 6592, and GPS only satellite systems are supported by default on these systems.

Because MT3332 can use GPS+GLONASS or a combination of GPS+BEIDOU, some customers will choose to use MT3332 as the positioning chip.

Then you may encounter some problems when porting MT3332 on these platforms, please refer to

FAQ06250 How to transplant MT3332

FAQ12394 Problems encountered in debugging MT3332

FAQ11721 MT3332 gps_tcxo_type instruction

Through these three FAQs, you can definitely get 3332 to work normally.

**Ø 8.4 ) If I want to analyze gps performance , what log can I refer to ?**

Please refer to the following:

FAQ07950 How to analyze GPSLog

**Ø 8.4 ) How to debug GPS without screen ?**

Please refer to the following:

FAQ13935 GPS problem debugging-debugging GPS without screen

MAUI GPS special knowledge

**Ø 9.1 ) What GPS- related items are in the Makefile ?**

GPS_SUPPORT: NONE, MT3336, MT3332. Only one can be selected.

GPS_HS_SUPPORT: FALSE, TRUE. Only one can be selected.

AGPS_SUPPORT: NONE, BOTH, CONTROL_PLANE, USER_PLANE. Only one can be selected.

GPS_ADAPTOR_SUPPORT: FALSE, TRUE. Only one can be selected.

GPS_LLE_SUPPORT: FALSE, TRUE. Only one can be selected.

  

Special reminder that the settings in these makefiles cannot be modified by themselves.

If you want to change the feature, you must apply for a flavor build.

MT3336 is a GPS only system.

-   MT3332 is a GNSS system.
-   On IOT , you can choose to support GPS+GLONASS , or a combination of GPS+BEIDOU .  
    

GPS+GLONASS is supported by default, and you can switch to GPS+BEIDOU.

-   In the phone product line, only GPS+GLONASS can be supported .
-   GPS_HS_SUPPORT is a function of Hotstill , if you open it, you need to reserve 1MByte ROM . Users can choose to open, but pay attention to user space.
-   GPS_ADAPTOR_SUPPORT=TRUE means it is an IOT product and supports EPO .
-   For GPS_ADAPTOR_SUPPORT and GPS_LLE_SUPPORT only IOT only open products.
-   AGPS_SUPPORT is currently only supported on IOT and not supported on the phone .
-   Because EPO is not supported by default on PHONE , if the customer wants to support EPO , the client needs to perform porting according to FAQ13228 . MT3326 has been phased out and does not support EPO .
-   GPS needs ROM : 800KB ; Hotstill needs ROM : 900KB ; AGPS : 200KB ; EPO needs 270KB .
-   AGPS and Hotstill can be supported on Phone ; EPO and Hotstill are supported on IOT .  
    

**Ø 9.2 ) What does FULL start , COLD start , WARM start and HOT start mean?**

The most important auxiliary information in the positioning process includes time, location, and ephemeris.

FULL start: There is no auxiliary information. It is equivalent to the scenario where the end user uses the positioning application after buying the mobile phone for the first time.

COLD start: There is time for auxiliary information, the end user will not encounter this scene.

WARM start: There is time and location assistance information. The end user's current positioning is more than 2 to 4 hours away from the last positioning.

HOT start: With all the auxiliary information, the end user's positioning is less than 2 to 4 hours from the last positioning.

So the scene that end users often encounter is WARM/HOT start.

**Ø 9.3 ) What is the TTFF of various startup methods ?**

The results of TTFF are strongly related to the test environment, test methods, and the GPS performance of the hardware.

The data given by MTK is based on the open sky environment, there are 6 satellites SNR "40db.

FULL start TTFF: less than 50s.

COLD start TTFF: less than 40s.

WARM start TTFF: less than 35s.

HOT start TTFF: less than 5s.

**Ø 9.4 ) What are the auxiliary positioning technologies?** Please refer to: FAQ13221 GPS problem debugging--not working

Or please refer to: FAQ13274 [GPS] GPS problem debugging--not working

**Ø 9.6 ) How to debug the problem that GPS can not find satellites?**

Please refer to: FAQ13222 GPS problem debugging-can't find satellite

Or please refer to: FAQ13275 [GPS] GPS problem debugging-satellites cannot be found

**Ø 9.7 ) How to debug the slow GPS search problem?**

Please refer to: FAQ13223 GPS problem debugging-slow positioning

Or please refer to: FAQ13276 [GPS] GPS problem debugging-slow positioning

**Ø 9.8 ) How to check the GPS search status with miniGPS tool ?**

Please refer to: FAQ13224 GPS problem debugging-Minigps joint debugging

Or please refer to: FAQ13277 [GPS] GPS problem debugging-Minigps joint debugging

**Ø 9.9 ) How to develop GPS on MAUI ?**

Please refer to: FAQ13225 GPS problem debugging-how to develop GPS on MAUI

Or please refer to: FAQ13278 [GPS] GPS problem debugging--how to develop GPS on MAUI

**Ø 9.10 ) How to test GPS Performance on MAUI ?**

Please refer to: FAQ13226 GPS problem debugging-how to test GPS Performance on MAUI

Or please refer to: FAQ13279 [GPS] GPS problem debugging--how to test GPS Performance on MAUI

**Ø 9.11 ) How to debug the problem of EPO download failure?**

Please refer to: FAQ13227 GPS problem debugging--how to confirm that EPO has downloaded successfully

Or please refer to: FAQ13280 [GPS] GPS problem debugging--how to confirm that EPO has downloaded successfully

**Ø 9.12 ) How PHONE on the EPO to support them?**

Please refer to: FAQ13228 GPS problem debugging-how to support EPO on PHONE

Or please refer to: FAQ13282 [GPS] GPS problem debugging--how to support EPO on PHONE

**Ø 9.13 ) What is the startup process of GPS ?**

Please refer to: FAQ12085 GPS startup process

Or please refer to: FAQ13284 [GPS] GPS startup process

**Ø 9.14 ) What commands does GPS support ?**

Please refer to: FAQ12093 How to issue a command to clear auxiliary information to GPS

Or please refer to: FAQ13285 [GPS] How to issue a command to clear auxiliary information to GPS

**Ø 9.15 ) How to save the gps log in the local device?**

Please refer to: FAQ13242 GPS problem debugging-how to save GPSLog to the device

Or please refer to: FAQ13281 [GPS] GPS problem debugging--how to save GPSLog to the device

**Ø 9.16 ) What log does AGPS certification need to capture ?**

Instrument log, gps debug log, cather log. For capturing gps debug log, please refer to:

FAQ04657 GPS Debug Log User Manual and log files needed for AGPS debug

**Ø 9.17 ) Does MT6261M support GPS chip MT3332/MT3336 ?**

not support.

Because MT3332/MT3336 needs 32K to work, 6161M cannot output 32K.

**Ø 9.18 ) Does MAUI support GPS chips such as MT3333/MT3339/MT3329/MT3337 ?**

Not supported by default.

But because these GPS chips themselves can highlight the standard NMEA sentence data processed by GPS through UART. Therefore, on MAUI, customers only need to receive GPS data through the serial port.

But it is still recommended to use our supported MT3336/MT3332 on MAUI.

**Ø 9.19 ) How to synchronize GPS to system time?**

The positioning must be completed.

The data in NVRAM_EF_GPS_MMI_SETTING_DATA_LID corresponds to time_sync=1.

**Ø 9.20 ) How to switch MT3332 to GPS+BEIDOU mode?**

Please refer to the Driver_allinone_for_MT6261x_MT250x.pptx document.

**Ø 9.21 ) Can EPO be downloaded via BT ?**

Can. You need to install Smartdevice APP on your smartphone.

**Ø 9.22 ) What is the power consumption of GPS on Tracker and IOT ?**

The power consumption is related to the gps data reporting cycle. The period value is greater than 1s, and there is no upper limit.

When the gps data reporting period is greater than 1s and less than 5min, it will enter a low-power mode, that is, work-sleep mode, WORK: 16.59ma; SLEEP: 2.43 ma

When the gps data reporting period is greater than 5 minutes, it will enter a low power consumption mode, that is, an ON-OFF mode.

ON: 15.545ma; OFF: 0.602 ma.

**Ø 9.22 ) How to verify the auxiliary effect of EPO ?**

The most improved EPO is warm start, which can be improved to the same effect as hot start.

To clear auxiliary data, please refer to: FAQ12093 How to issue a command to clear auxiliary information to GPS

Or please refer to: FAQ13285 [GPS] How to issue a command to clear auxiliary information to GPS

**Ø 9.23 ) Watch Tracker on gps relevant feature presentation?**

You can refer to "FAQ13861 [GPS] GPS problem classification-Watch_Tracker_GPS_Feature_Introduction", or include the introduction of low power consumption features.

**Ø 9.24 ) How to develop GPS application VXP ?**

You can refer to "FAQ13860 [GPS] GPS problem classification-how to compile GPS VXP", here will include the analysis of GPS data.
