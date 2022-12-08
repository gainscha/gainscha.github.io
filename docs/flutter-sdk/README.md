# 标签打印SDK for Flutter

标签打印机sdk

支持多个系统平台
1. Android
2. IOS
3. Windows
4. Linux
5. MacOS

支持多种连接方式
1. 蓝牙
2. USB
3. WIFI
4. 串口

支持多种打印机指令集
1. TSPL标签指令
2. ESC票据指令
3. CPCL面单指令
4. ZPL斑马指令

## 1.连接打印机

### 1.1 引入SDK依赖

```yaml
dependencies:
  label_print_sdk: ^1.0.0
```

### 1.2 搜索打印机

```dart
/// 扫描USB设备
void _scanUsbPrinter() async {
  final Printer _printer = Printer.instance;
  var deviceList = await _printer.scanUsbDevice();
  deviceList?.forEach((device) {
    print("usbDevice ${device.name}");
  });
  if (deviceList != null) {
    setState(() {
      _deviceList.addAll(deviceList);
    });
  }
}
```
### 1.3 监听打印机连接状态
```dart
@override
void initState() {
  super.initState();
  //监听打印机连接状态变化
  _subscription = _printer.listenPrinterConnectState().listen((device) {
    setState(() {
      _deviceListConnected.clear();
      if (device != null && device.isNotEmpty) {
        _deviceListConnected.addAll(device);
        _printerDevice = _deviceListConnected.first;
      }
    });
  });
}

@override
void dispose() {
  super.dispose();
  _subscription?.cancel();
}
```
### 1.4 获取已连接的打印机
```dart
void _test() {
  _printer.getConnectedPrinters().then((value) {
    if (value != null && value.isNotEmpty) {
      setState(() {
        _deviceListConnected.addAll(value);
      });
    }
  });
}
```

## 2.标签内容设计

这里以TSPL标签指令为案例：

```dart
/// 打印
void _print() {
  int y = 0;
  Tspl cmd = Tspl(60, 40)
    ..addText(x: 0, y: y, text: "测试文本")
    ..addText(x: 0, y: y += 30, text: "测试文本", fontMultiply: 2)
    ..addBarcode(x: 0, y: y += 60, text: "12345678", height: 70)
    ..addQrcode(x: 0, y: y += 80, text: "qrcode")
    ..addLine(0, y += 80, 300, 5)
    ..addBox(0, y += 10, 300, y, 3)
    ..print(1);
  if (_printerDevice != null) {
    _printer.write(_printerDevice!, cmd.toBytes());
  }
}
```