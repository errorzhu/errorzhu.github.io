# 树莓派4b 如何查询gpio和设备文件对应

## 查询串口对应地址

```
 dmesg |grep tty
```

## 查询gpio对应地址

```
 cat /sys/kernel/debug/pinctrl/fe200000.gpio-pinctrl-bcm2835/pinmux-pins
```

## 查询串口输入输出

```
 cat  /proc/tty/driver/ttyAMA
```