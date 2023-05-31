# ov5640-VGA

12010508 Yuxiao Hua

## System description

### Function

In this project, I am going to design a system to implement the following functions: use OV5640 to read picture from the environment, and display the image captured from the camera in real time. 

### Specification

resolution: 640x480
frame rate: 60 fps
color standard: RGB444

### Development environment

language: VHDL
software: Vivado 2022.2
protocol: I2C

### Extension

If time permits, I am planning to attach additional functions to the system. For example, digit and letter recognition.

### Schedule

Since my teammate refused to cooperate, I am responsible for the whole project.

- **week 9**: purchase necessary items for the porject
- **week 10 and 11**: equipe myself with necessary knowledge of OV5640, I2C, VGA and RAM
- **week 12 to 14**: deal with code matters and debug
- **week 15**: final check and extension functions

## Top module 

### Block diagrams

The whole system works following the order below: 

Module **clk_wiz_0** serves as a  clock frequency divider, which provides three outputs: `clk_24m`, `clk_25m` and `clk_50m`. Signal `rst_n` controls the whole system to start or stop, once set to low, module **power_on_delay** outputs a specific signal sequence `pwdn` to module **ov5640_capture** inder to start up OV5640. After completing the after process, the module **I2C_com** start to configure the 250 registers of OV5640. Once finished, the camera start to capture image data with clk `xclk`. The control signal for camera to work properly is 'vsync` and `href`, whos woking principle will be talked about later in the report. The camera module converts the analog signal to digital signal of formation RGB444. The `addr` and `d[7:0]` define the data transfering rules. The image data is kept in module **blk_men_gen_0**, which is a type of RAM. With clock signal synchronized, module **ov5640_vga** outputs the RGB data in RAM. We can see the image displayed on the screen in real time.

![image](https://github.com/HuaYuXiao/ov5640-VGA/assets/117464811/efe4dbf8-df4c-4290-8a72-6d64c38d3f1c)

### Global inputs and outputs

```VHDL
entity top is
  Port ( 
//clock signal used for system synchronization    
    sys_clock: in std_logic;
//reset signal used for resetting the system    
    rst_n:in std_logic;
//pixel clock signal from the OV5640 camera module    
  pclk:in std_logic;
//vertical sync signal from the OV5640 camera module  
  vsync:in std_logic;
//Input horizontal sync signal from the OV5640 camera module  
  href:in std_logic;
//data bus carrying the pixel data from the OV5640 camera module  
  d:in std_logic_vector (7 downto 0 );
//signal indicating the completion of camera configuration  
  config_finished:out std_logic;
//signal used for the I2C SIOC (Serial Clock) line of the OV5640 camera module  
  sioc:out std_logic;
//signal used for the I2C SIOD (Serial Data) line of the OV5640 camera module  
  siod:inout std_logic;
//signal used for resetting the OV5640 camera module  
  reset:out std_logic;
//signal used for controlling the power-down mode of the OV5640 camera module  
  pwdn:out std_logic;
//pixel clock signal provided to the OV5640 camera module  
  xclk:out std_logic;
//horizontal sync signal for VGA display  
  vga_hsync:out std_logic;
//vertical sync signal for VGA display
  vga_vsync:out std_logic;
//signal carrying the red color component for VGA display  
  vga_r:out std_logic_vector ( 3 downto 0);
//signal carrying the green color component for VGA display  
  vga_g:out std_logic_vector( 3 downto 0);
//signal carrying the blue color component for VGA display
  vga_b:out std_logic_vector( 3 downto 0)
  );
end top;
```

## Pmods devices

### OV5640

**cost**: 168 yuan

The OV5640 is a high-performance 1/4-inch CMOS image sensor designed and manufactured by OmniVision Technologies. It is an image sensor widely used in digital cameras, smartphones and embedded systems.

![th](https://github.com/HuaYuXiao/ov5640-VGA/assets/117464811/3363c16f-faf1-4a56-ac5d-2564dfc40d14)

The following is the port definition of the module:

```VHDL
entity ov5640_capture is
    port (
//Pixel Clock is the input signal used to drive the camera's image acquisition and data transmission.
        pclk  : in   std_logic;
//The camera vertical sync signal (Vertical Sync) is an input signal used to indicate the camera's vertical sync moment, that is, the frame start and end of the image.        
        vsync : in   std_logic；
Camera line synchronization signal (Horizontal Reference), is an input signal used to indicate the camera's line synchronization moment, that is, the line start and end of the image.        
        href  : in   std_logic;
The camera data bus (Data Bus) is an input signal used to transmit the pixel data collected by the camera.        
        d     : in   std_logic_vector ( 7 downto 0);
Output signal, which is an 18-bit signal that transmits the address of the data read from the camera.        
        addr  : out  std_logic_vector (17 downto 0);
Output signal, which is a 12-bit signal that transmits pixel data read from the camera.        
        dout  : out  std_logic_vector (11 downto 0);
Output signal, indicating Write Enable, which indicates data reading operations on the camera.        
        we    : out  std_logic
    );
end ov5640_capture;
```

The figure below shows the structure of OV5640. 

![image](https://github.com/HuaYuXiao/ov5640-VGA/assets/117464811/79b3e73b-d511-4826-b136-1aa64954958d)

Combining the above structure diagram, the working principle of the module is as follows:

- **Array of Light Sensitive Elements**: The OV5640 contains an array of light sensitive elements, typically manufactured using CMOS (Complementary Metal Oxide Semiconductor) technology. Each pixel in the array corresponds to a point in the image and is able to perceive the intensity of the light.

- **Pixel reading and amplification**: When the camera receives a pixel clock (PCLK) signal, each pixel is read line by line. During reading, pixel values are amplified by amplification circuitry to enhance signal quality and image detail.

- **Line sync and frame sync signals**: The OV5640 uses line sync signals (href) and frame sync signals (vsync) to synchronize lines and frames of an image. The row-sync signal represents the beginning and end of a row of pixel data, while the frame-sync signal represents the beginning and end of a framed image.

![image](https://github.com/HuaYuXiao/ov5640-VGA/assets/117464811/3e540f20-227d-4e37-8f3c-ebb8c2863f97)

- **Data transfer**: Readout pixel data is transmitted via the data bus (d) to an external device such as an FPGA, processor, or memory. Each pixel is typically represented by multiple bits, for example, each pixel in an RGB color image is typically represented by three components: red, green, and blue.

- **Control interface**: The OV5640 also provides a control interface that allows external devices to configure and control various parameters of the camera, such as image resolution, exposure time, white balance, etc., via I2C or other communication protocols.



### VGA screen

**cost**: 168 yuan














 Test results, analysis of performances, meeting of specifications and
limitations of the system.
