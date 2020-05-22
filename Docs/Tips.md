配置提示和建议
==================

下面是一些配置过程中的建议和提示，以帮助你更好的完成配置。

#### 测试是否带原生NVRAM，请在终端执行以下命令:

1. 添加一个测试变量: sudo nvram TestVar=HelloWorld
2. 然后重启运行: sudo nvram -p | grep 'TestVar'
3. 检测是否成功后删除该测试变量（sudo nvram -d TestVar）

#### Boot参数说明

> `dar=0` 提供针对VT-d的额外保护并删除ACPI中的DAMR表，推荐在OC中使用DisableIoMapper
>
> `debug=0x100` 若重启时发生内核恐慌，启用此项来查看原因
>
> `keepsyms=1` 可选项，打印内核崩溃的日志信息



#### 大体思路

如果你已有Clover引导环境，我建议参考你的Clover修补方法来配置OpenCore。
虽然两者有区别，但一些补丁实现方式差不多，这样可以让你的配置步骤更有条理。

- OpenCore官方不建议使用更名补丁，除非必须使用的情况。因为新版Whatevergreen等驱动已经自带常用更名并注入一些必要设备，所以Clover下的一些常用二进制更名补丁(GFX0->IGPU，HDAS->HDEF，HECI->IMEI等)也不再需要。能使用SSDT注入的都优先考虑用SSDT注入，尽管某些功能在Clover下可以直接勾选且很方便)，但制作一个兼容性好的SSDT补丁Clover和OpenCore都可以使用，一次制作，两处使用。
- 尽可能采用添加设备属性(DeviceProperties)对PCI设备打补丁。
  例如通过Properties方法注入IMEI，PciRoot(0x0)/Pci(0x16,0x0):device-id, 3A1E0000， 当Properties方法不奏效时或者其他原因时，再采用对设备或者方法更名以及hotpatch的SSDT文件对其实施定制补丁。
- 显卡大多数情况下使用Whatevergreen即可解决。
- 声卡配合AppleALC驱动方法有很多种(Clover,DSDT,设备属性)，如果采用DSDT注入的话，请禁用Clover相关选项(Devices-> Audio-> Inject = NO)。个人建议用设备属性这种方式来注入，并且这也是OpenCore推荐采用的方式。
- SATA类型的SSD若要开启TRIM，建议使用终端命令"sudo trimforce enable"来启用，不建议使用CloverKext补丁(Enable TRIM for SSD)或OpenCore的相关项(ThirdPartyTrim)。
- USB相关，因为苹果原生EC控制器在DSDT中就叫EC，是否使用重命名补丁请先查看原机DSDT的EC控制器名字，可能叫H_EC或EC0，方法是搜索DSDT中"PNP0C09"的设备，如果该设备的"_STA"返回值为零(Zero)，直接添加SSDT-EC-USBX.aml文件(注入EC控制器和电源管理)，反之则应在SSDT中将EC禁用，USB电源问题使用iPad测试比较方便。
- 强烈推荐使用我定制的USB端口补丁(15port版)，以避免缓冲区溢出。未带15port的驱动为全部端口可用，需配合USB端口限制补丁才能使用，自定义时请注意info.plist中IOMatchName项的名称。
- 网卡，使用相应的kext网卡驱动即可。

- 注意：在使用OpenCore引导双系统时，如果遇到Windows激活环境失效的问题，可尝试在配置文件中使用主板的UUID作为SystemUUID。通常主板的UUID在BIOS中就可以看到，如果BIOS中看不到，可以通过传统方式启动到Windows（不能是OC引导），然后在命令行中查看，打开cmd，输入wmic并回车，会看到提示符变成了 wmic:root\cli>，再输入csproduct list full，得到内容如下：

  ```
  C:\Users\JFZ>wmic
  wmic:root\cli>csproduct list full
  
  Description=Coputer System Product
  IdentifyingNumber=Default string
  Name=Prime B250M-A
  SKUNumber=
  UUID=00000000-0000-0000-0000-XXXXXXXXXXXX
  Vendor=ASUS.....
  Version=1.0
  ```

  将上面的UUID填入config.plist中的Platforminfo>Generic>SystemUUID即可。
