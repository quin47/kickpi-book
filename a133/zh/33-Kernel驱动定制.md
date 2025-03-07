# Kernel 驱动定制

### UART

K5C 拓展引脚中如PL2，PL3默认配置成GPIO，可复用成UART

**快速确认pin功能**

```
$ ls longan/out/a133/c3/android/.board.dtb.dts.tmp
```

> 此文件需要编译后才会生成

从.dts.tmp文件中可以确认UART7是PL2，PL3配置，配置信息类似如下

```
s_uart0_pins_a: s_uart0@0 {
    allwinner,pins = "PL2", "PL3";
    allwinner,function = "s_uart0";
    allwinner,muxsel = <2>;
    allwinner,drive = <1>;
    allwinner,pull = <1>;
};
...
s_uart0_pins_b: s_uart0@1 {
    allwinner,pins = "PL2", "PL3";
    allwinner,function = "io_disabled";
    allwinner,muxsel = <7>;
    allwinner,drive = <1>;
    allwinner,pull = <1>;
};
...
uart7: uart@07080000 {
   compatible = "allwinner,sun50i-uart";
   device_type = "uart7";
   reg = <0x0 0x07080000 0x0 0x400>;
   interrupts = <0 112 4>;
   clocks = <&clk_suart>;
   pinctrl-names = "default", "sleep";
   pinctrl-0 = <&s_uart0_pins_a>;
   pinctrl-1 = <&s_uart0_pins_b>;
   uart7_port = <7>;
   uart7_type = <2>;
   status = "disabled";
};
```

**dts配置UART**

单个引脚仅能用于一个功能，取消PL2，PL3 的GPIO配置

```diff
--- a/longan/device/config/chips/a133/configs/c3/kickpi-k5c.dts
+++ b/longan/device/config/chips/a133/configs/c3/kickpi-k5c.dts
@@ -1409,17 +1409,17 @@
                                linux,default_trigger = "default-on";
                        };
 
-                       PL2 {
-                               label = "PL2";
-                               gpios = <&r_pio PL 2 1 0 1 0>;
-                               linux,default_trigger = "default-on";
-                       };
-
-                       PL3 {
-                               label = "PL3";
-                               gpios = <&r_pio PL 3 1 0 1 0>;
-                               linux,default_trigger = "default-on";
-                       };
+                       // PL2 {
+                       //      label = "PL2";
+                       //      gpios = <&r_pio PL 2 1 0 1 0>;
+                       //      linux,default_trigger = "default-on";
+                       // };
+
+                       // PL3 {
+                       //      label = "PL3";
+                       //      gpios = <&r_pio PL 3 1 0 1 0>;
+                       //      linux,default_trigger = "default-on";
+                       // };
 
```

打开UART7

```diff
--- a/longan/device/config/chips/a133/configs/c3/kickpi-k5c.dts
+++ b/longan/device/config/chips/a133/configs/c3/kickpi-k5c.dts
+&uart7 {
+    status = "okay";
+};
```

**测试UART**

首先连接好硬件，RX连接TX，GND连GND

板端可通过microcom进行测试

```
microcom -s 115200 /dev/ttyS2
```

另一端可通过 Mobaxterm 或其他串口工具进行测试



