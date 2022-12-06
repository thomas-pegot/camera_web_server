# Camera webserver with motion

## On going Changes

DCT Sign-Only Correlation with Application to Image Matching and the Relationship with Phase-Only Correlation ([DOI:10.1109/ICASSP.2007.366138](https://www.researchgate.net/publication/224711136_DCT_Sign-Only_Correlation_with_Application_to_Image_Matching_and_the_Relationship_with_Phase-Only_Correlation))

Configuration : 
```c
#define DCT_FILTER 0
#define AC_OPTIM 0
#define PHASE_DCT 1
```

|Motion without output| Motion with output |
|---|---|
|![phase](data/motion_b.gif) | ![motion](data/motion.gif)|
|`#define PHASE_DCT 3`|`#define PHASE_DCT 2`|

**Interpretation**: Motion are created from the DC component only so we can't capture motion where there is edges already. The advantage though is that you don't have those horizontal lines interfering. Atm I can't fix the issue related of background edges interfering the motion.


|DCT-sign DC + All AC| DCT-sign DC only |
|---|---|
|![phase](data/DCT-sign_DCplusAC.gif) | ![motion](data/DCT-sign_DConly.gif)|
|`#define PHASE_DCT 1`|`#define PHASE_DCT 1` + manual changes|
|Edges of objects are more detailed because we have all frequencies.|Horizontal line artifacts disappear. But loose edges details.|

**Interpretation**: The process of returning moving object is done by differentating the background (DCT-sign DC only) from the current image (DCT-sign DC only). However since the background DC are the same values as the moving object, the differentiate would be 0 and therefore won't considerate as moving object. Atm I can't find any solution to fix this.


## Changes
- esp32motion : enhanced EPZS algorithm accuracy. Now it is possible to choose FFMPEG or MPEG4_AVC version by changing `FFMPEG` value in `esp32-motion/epsz.c` 
- added JPEG decoder :
  - Now it won't load the library from the internal ROM/jpgd.h but instead a custom `jpgd.h` library in development.
  - `jpgd.h` can handle new features, at the moment : 
    -   `jpg2gray` : convert directly during decoding to monochrome
    -   `jpg2gray_filtered` : during conversion apply a median filter to remove horizontal line artifacts. (cf .gif below)
    -   developped a DCT-phase filter which can be usefull for motion detection
    -   developped a DCT based denoiser
  -   Removed upscaling (too slow)
  -   GUI added a test "Motion Algorithm" which is just the capture converted to gray. It shows the decoder latency and the effect of artifacts filtering.   

### `jpg2gray_filtered`
![jpg2gray_filter](data/jpg2gray_filter.gif)


### Note 
 - DCT denoiser can be enable/disable in `tjpgdcnf.h` under `DCT_FILTER`
 - DCT phase and modulation can be enable/disable in `tjpgdcnf.h` under `PHASE_DCT`
## Old Results

| algo  | demo  | input size | time |
|---|---|---|---|
|  **lucas kanade** |  ![lucaskanade_demo](data/lucaskanade_demo.gif)   | 160 x 120 |  600ms (+130ms display time)
| **block matching ARPS** |  ![arps_demo](data/arps_demo.gif) | 640 x 480 | 130ms (no motion) to 380ms (motion)  (**+2sec display**) with 8x8 MB search 9|
| **block matching EPZS** | ![epzs_demo](data/epzs_demo.gif) | 640 x 480 |  150ms to 380ms (**+2sec display**) with 6x6 MB search 9|

Note : Block matching takes longer to display (2sec per frame) because of upscaling to input image size for a nicer display. That's why we see so much latency.


## Usage block matching

![details](data/view-detailed.png)

## Preparation

To run this example, you need the following components:

* An ESP32 Module: Either **ESP32-CAM**.
* A Camera Module: Either **OV2640**.

## Quickstart with PlatformIO + VScode

 - Press F1 (_Show All Commands_) and type `PlatformIO: Open PlatformIO Core CLI`. It will open a terminal and load `pio`. From there run the command :
   - `pio run -t menuconfig -e release` : Will load release config file  _sdkconfig.release_ (-O2 optim, IRAM optimisation). This mode is specific for serial device since it can't debug.
   - `pio run -t menuconfig -e debug` : Will load debug config file  _sdkconfig.debug_ (-Og debug, Flash instead of IRAM, reduced CLCK speed). This mode is specific for JTAG/ESP-Prog device.
   - Under `Camera Web Server ---> Wifi Settings` you can change WiFi configuration (STA and AP)
 - `pio run -t build -e release` or `debug`
 - `pio run -t upload -e release`  or `debug`
 - Follow from step 4. in next section



## Quick Start

After you've completed the hardware settings, please follow the steps below:

1. **Connect** the camera to ESP32 module. For connection pins, please see [here](https://github.com/espressif/esp-who/blob/master/docs/en/Camera_connections.md)
2. **Configure** the example through `idf.py menuconfig`;
3. **Build And Flash** the application to ESP32;
4. **Open Your Browser** and point it to `http://[ip-of-esp32]/`;
5. **To Get Image** press `Get Still` or `Start Stream`;
6. **Use The Options** to enable/disable Motion detection, Filtering and more;
t. **View The Stream**  in a player like VLC: Open Network `http://[ip-of-esp32]:81/stream`;

For more details of the http handler, please refer to [esp32-camera](https://github.com/espressif/esp32-camera).


## References

This code is revisited from [https://github.com/espressif/esp-who](https://github.com/espressif/esp-who) .

## License
[MIT](https://choosealicense.com/licenses/mit/)
