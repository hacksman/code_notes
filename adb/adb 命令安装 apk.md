1. 通过`adb devices -l` 检查是否手机与电脑正常连接。若正常连接会出现类似以下信息
```shell
List of devices attached
PBV5D2524C8EF75D       device usb:340787200X product:EVA-AL10 model:EVA_AL10 device:HWEVA transport_id:1
```
2. 通过`adb install ./xx/xx.apk` 安装即可