### Linux连接蓝牙键盘

今天尝试了一下用蓝牙连接我的小小树莓派，遇到了一些问题，这里总结一下解决方案

1. 打开蓝牙控制器

   ```
   sudo bluetoothctl
   ```

2. 扫描蓝牙设备，这时候保证蓝牙设备已经打开

   ```
   [bluetooth]# scan on
   
   Discovery started
   [CHG] Controller 5C:F3:70:6B:57:60 Discovering: yes
   ```

3. 查看设备

   ```
   [bluetooth]# devices
   [NEW] Device 08:DF:1F:A7:B1:7B Bose Mini II SoundLink
   ```

4. 发现设备的mac地址之后，请求连接

   ```
   [bluetooth]# pair 08:DF:1F:A7:B1:7B
   
   Attempting to pair with 08:DF:1F:A7:B1:7B
   [CHG] Device 08:DF:1F:A7:B1:7B Connected: yes
   [CHG] Device 08:DF:1F:A7:B1:7B UUIDs:
       0000110b-0000-1000-8000-00805f9b34fb
       0000110c-0000-1000-8000-00805f9b34fb
       0000110e-0000-1000-8000-00805f9b34fb
       0000111e-0000-1000-8000-00805f9b34fb
       00001200-0000-1000-8000-00805f9b34fb
   [CHG] Device 08:DF:1F:A7:B1:7B Paired: yes
   Pairing successful
   ```

5. 之后再信任一下该设备，确保重启之后不需要重新连接

   ```
   [bluetooth]# trust 08:DF:1F:A7:B1:7B
   
   [CHG] Device 08:DF:1F:A7:B1:7B Trusted: yes
   Changing 08:DF:1F:A7:B1:7B trust succeeded
   ```

6. 之后再手动连接一下蓝牙设备

   ```
   [bluetooth]# connect 08:DF:1F:A7:B1:7B
   
   Attempting to connect to 08:DF:1F:A7:B1:7B
   ```

   [返回主页](../index.html)