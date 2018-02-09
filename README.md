# 这是ATMEL 基于ATmega328P和NRF24L01无线模块的遥控接收机
## 特性:
- 可使用Arduino IDE编程
- 四通道PWM输出（可在设定文件内更改输模式）
- 内置TB6612FNG双直流电机驱动芯片
- 串口输出接口
- I2C 接口 (支持MPU-6050)
- ICSP 接口 for programming with Arduino IDE (use "TheDIYGuy999 328P" board definition and set brownout voltage to 1.8V)
- 内置贴片NRF24L01
- 尺寸38*26mm（不计贴片无线模块天线部分）
- Eagle files are also provided, if you want to build your own receiver!

See: https://www.youtube.com/playlist?list=PLGO5EJJClJBCjIvu8frS7LrEU3H2Yz_so

New in V 1.3:
- Vehicle configuration is now stored in "vehicleConfig.h", so the main code always stays the same.
- Crude frequency hopping, if bad receiving quality is detected (needs to be improved).

New in V 1.4:
- 10 vehicle addresses instead of 5
- More vehicle configuration options
- The "BACK" button on the transmitter can now be used (if the menu screen is not displayed on the OLED) as a momentary button, for example for a horn on the vehicle. It is sent as "data.momentary1"
- SimpleTimer library replaced with my own code, which uses significantly less memory

New in V 1.5:
- Added steering mixer for semi caterpillar and caterpillar vehicles such as tanks, diggers etc.
- Channel 3 is throttle, channel 1 is steering
- More vehicle configuration options in vehicleVonfig.h
- The left caterpillar is motor 2, the right is motor 1
- Steering servo 1 can be used in parallel (for semi caterpillar vehicles or vehicles with electronic differential)

New in V 1.6:
- Added indicator lights - wired to A4 (SDA) / A5 (SCL) ports
- enabled and disabled left / right by channel 4 left / right
- disabled by turning back the steering wheel (channel 1) - just like a real car
- hazard lights (all indicators flashing), if no connection to the transmitter

New in V 1.61:
- Battery cutoff bug fixed

New in V 1.7 (Support for board revision 1.3):
- **** TB6612FNG library update to V1.1 is required! ****
- motor driver pinout has changed in board V1.3. So we are now able to adjust the PWM frequency of motor2. This allows smooth motor operation without the typical PWM whining.
- vehicleConfig.h changed: "boardVersion" and "HP" variables added. They allow to stay compatible with older board versions. Just select the correct configuration.
- The new "HP" (High Power) board version has only one motor driver channel, because both TB6612FNG channels are wired in parallel. It's able to supply 2.4A (average) / 6.4A (peak) so we can use bigger motors. This is useful for models, which use a steering servo and don't need a second motor channel.'
- automatic radio re-initialisation after signal timeout

New in V 1.71
- Battery monitoring and cutoff triggering improved. Under load, 0.3V are now added to the battery voltage to compensate the voltage drop. This allows to set the cutoff voltage to 3.6V, even with bigger loads.
- Vehicle #7 added.

New in V 1.8
- New "engineSound" variable in "vehicleconfig.h". True means, that the servo 3 pulse signal can be switched on and off, using the  channel 2 joystick. This switches the engine sound on my "Rc Engine Sound" unit on and off.

New in V 1.9
- New vehicle specific configuration section in vehicleConfig.h. The configuration is now done entirely during compilation time.

New in V 2.0
- vehicle Config.h: New vehicleType 3 (forklift) added. Motor 2 is used as lifting motor in this case. A servo is required as steering motor.
- Beacon lights can be wired to servo channel 4. Select "beacons = true" in this case. The beacons are flashing during vehicle movement and are switching off, after the vehicle did not move for 10s.

New in V 2.1
- Sounds for a STAR WARS R2-D2 robot can now be generated on servo channel 3
- "toneOut" variable added to vehicleConfig.h. True means, that servo 3 channel is generating sounds instead of a servo signal.
- Do not enable "toneOut" and "engineSound" at the same time.
- Note: the Arduino tone() function is blocking. So don't play sounds while driving ;-)

New in V 2.2
- **** TB6612FNG library update to V1.2 is required! **** https://github.com/TheDIYGuy999/TB6612FNG
- **** The PID library is required! **** https://github.com/br3ttb/Arduino-PID-Library/
- Support for self balancing (inverted pendulum, segway) vehicles. See: https://www.youtube.com/watch?v=zse9-l2Yo3Y)
- A MPU-6050 accelerometer & gyro module is used for the balancing data communication via SDA & SCL
- Note, that the receiver will not work, if your vehicleType is 4 (balancing) and no MPU-6050 is connected!
- Do not enable "balancing" and "indicators" at the same time.

New in V 2.21
- Gyro drift calibration bug fixed
- Changed the balancing controller refresh rate to 125Hz to reduce jitter on a 8MHz MCU. Reason: one program loop takes 4 - 7ms, so we can't use a time base of 4ms (=250Hz).

New in V 2.3
- MRSC (Micro RC Stability Control) added. This functionality is similar to the Traxxas TSM or ESC. See: https://www.youtube.com/watch?v=IPve7QpdLBc&t=5s
- The steering servo angle is compensated in accordance with the speed, the requested steering angle and the measured yaw rate, which is coming from the MPU-6050
- Basically the mrsc() function was added. The required yaw rate is already calculated in the balancing.h, which is already existing for self balancing robots
- If you want to use this functionality, you have to set the vehicleType to 5 in the vehicleConfig.h. Please note, that the receiver will not work, if no MPU-6050 is connected and the vehicleType is set to 4 or 5!

New in V 2.31
 - MRSC gain factor now depending on the potentiometer 1 value on the transmitter
 - Allows to fine adjust the MRSC during driving, depending on the requirements of the street surface
 - Note, that you need to connect a potentiometer to the analog input A6 of your transmitter!
 
 New in V 2.32
 - Flickering headlights bug in vehicleType 5 fixed
 - The accelerometer vectors are not processed anymore in vehicleType 5. This is only required for the self balancing robot (vehicleType 4)
 
 New in V 2.4
 - The I2C bus is scanned for an MPU-6050 sensor during the setup() sequence. The vehicleType is automatically changed to 0 (car without MRSC), if no sensor is detected.
 - This means, that a car with vehicleType 5 (MRSC Stability Control) can now also be used without an MPU-6050 plugged in.
 - Note, that the self balancing mode (vehicleType 4) always requires an MPU-6050. Otherwise the receiver will not start.
 
 New in V 2.41
 - The "indicators" variable is now locked and can't be enabled, if the SDA and SCL pins are in use for the MPU-6050 sensor. This is the case in vehicleMode 4 and 5.
 
 New in V 2.5
 - The MRSC stability control is now also working for cars without a steering servo
 - Code was added to control a DC steering motor (as found in cheap RC cars) in accordance with the MPU-6050 yaw rate
 - This is for example very useful for Coke Can Cars. See: https://www.youtube.com/watch?v=jr5yYBal3vk

## Usage

See pictures
![](https://github.com/TheDIYGuy999/Micro_RC_Receiver/blob/master/1.jpg)
![](https://github.com/TheDIYGuy999/Micro_RC_Receiver/blob/master/2.jpg)
![](https://github.com/TheDIYGuy999/Micro_RC_Receiver/blob/master/3.jpg)

Receiver

![](https://github.com/TheDIYGuy999/Micro_RC_Receiver/blob/master/Micro_RC_Receiver.png)

MPU-6050 shield

![](https://github.com/TheDIYGuy999/Micro_RC_Receiver/blob/master/MPU-6050_Shield.png)

(c) 2016 - 2017 TheDIYGuy999
