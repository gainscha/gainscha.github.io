# 目录

 [TOC]

# 1. SDK介绍

此文档介绍佳博打印机SDK使用方式。SDK集成了打印机的多种连接方式，开发者可自己选择不同的方式连接打印机，也可自己实现。在SDK中APP与打印机连接的方式有5种，分别为：

1. 无线局域网（TCP）
2. USB接入
3. 串口
4. 蓝牙

开发者需要根据打印机所支持的方式选择合适的接入方式。

# 2. 打印机指令集

上位机与打印机通信的指令集有多种，开发者可以自己连接打印机后通过指令与打印机通信，此SDK已集成指令集接口。

打印机可支持如下的一种或多种指令集，根据实际情况选择使用。各指令集的功能如下：

| 指令集  | 介绍    |
| ---- | ----- |
| Tspl | 标签指令集 |
| Esc  | 票据指令集 |
| Zpl  | 斑马指令集 |
| Cpcl | 面单指令集 |

各指令的详细说明请查看佳博打印机编程手册

# 3. 快速接入

## 3.1 接入方式

在项目根目的的build.gradle文件中添加：

```gradle
repositories {
    maven {
        url  "http://118.31.6.84:8081/repository/maven-public/"
        allowInsecureProtocol true
    }
}
```

在需要使用SDK的模块的build.gradle中添加：

```gradle
dependencies {
    implementation 'com.gainscha:sdk2:2.0.0'
}
```

这里介绍如何使用SDK集成的连接接口连接并控制打印机

## 3.2. 搜索打印机

SDK提供了搜索打印机类(**PrinterFinder.java**)，实际开发中可根据需要选择是否使用。提供如下的功能：

```java
    /**
 * 获取所有连接方式的设备列表
 *
 * @param searchPrinterResultListener 查找到新设备回调
 */
public void searchPrinters(SearchPrinterResultListener searchPrinterResultListener);
/**
 * 获取可连接的设备列表
 *
 * @param connectType    连接方式
 * @param listener       查找到新设备回调
 */
public void searchPrinters(ConnectType connectType,final SearchPrinterResultListener listener);
/**
 * 停止搜索设备
 */
public void stopSearchDevice(Context context);
```

搜索结果回调

```java
/**
 * 扫描打印机设备的回调接口
 */
public interface SearchPrinterResultListener {
    /**
     * 搜索到蓝牙设备
     *
     * @param device 蓝牙设备
     */
    void onSearchBluetoothPrinter(BluetoothPrinterDevice device);

    /**
     * 搜索到USB设备
     *
     * @param device usb设备
     */
    void onSearchUsbPrinter(UsbPrinterDevice device);

    /**
     * 搜索到USB设备（配件模式）
     *
     * @param device usb配件
     */
    void onSearchUsbPrinter(UsbAccessoryPrinterDevice device);

    /**
     * 搜索网络设备
     *
     * @param device WIFI打印机
     */
    void onSearchNetworkPrinter(WifiPrinterDevice device);

    /**
     * 搜索到串口设备
     *
     * @param device 串口设备
     */
    void onSearchSerialPortPrinter(SerialPortPrinterDevice device);

    /**
     * 搜索完成
     */
    void onSearchCompleted();
}
```

使用示例

```java
// 搜索USB打印机
new PrinterFinder().searchPrinters(
    ConnectType.USB_DEVICE,
    new PrinterFinder.SimpleSearchPrinterResultListener() {
        @Override
        public void onSearchUsbPrinter(UsbPrinterDevice usbPrinterDevice) {
            super.onSearchUsbPrinter(usbPrinterDevice);
            // 搜索到USB打印机设备
        }
    });
```



## 3.3 监听打印机连接状态

```java
ConnectionListener listener = new ConnectionListener(){
    @Override
    public void onPrinterConnected(Printer printer) {
        //连接成功
    }

    @Override
    public void onPrinterConnectFail(Printer printer) {
        //连接失败
    }

    @Override
    public void onPrinterDisconnect(Printer printer) {
        //断开连接
    }
}
//添加连接状态监听回调（主线程回调）
Printer.addConnectionListener(listener)
//请在不需要回调时移除监听，避免内存泄漏
//如：在Activity的onCreate时添加监听，则在onDestroy时务必移除监听
Printer.removeConnectionListener(listener);
```

## 3.4 连接打印机

SDK支持多种方式连接到打印机，可以通过调用Printer类中的方法连接打印机。

```java
// 连接打印机
Printer.connect(printerDevice);
// 获取已连接的打印机列表
List<Printer> printers = Printer.getConnectedPrinters();
```

## 3.5 打印内容

连接到打印机后，可以调用如下方法发送数据到打印机，实现打印。

```java
//同步发送数据到打印机
//注意：使用Tspl、Esc、Cpcl、Zpl类生成打印数据
printer.print(byte[] data);

//异步发送数据到打印机，并读取打印机的响应内容（超时时间内首次接收到的数据）
printer.print(byte[] data, int responseTimeOut, PrinterResponse<byte[]> onResponse);

//异步发送数据到打印机，并读取打印机的响应内容
//responseUntilTimeout是否读取数据直到超时，当为true时会返回超时时间内读取的全部数据
printer.print(byte[] data, int responseTimeOut, boolean responseUntilTimeout, PrinterResponse<byte[]> onResponse);
```

# 4. 打印指令

### 4.1 标签指令（TSPL）打印

```java
public void print(Bitmap bitmap, int count){
    Tspl tsc = new Tspl();
    //设置大小
    tsc.addSize(60, 40);
    //设置打印速度
    tsc.addSpeed(SPEED.SPEED2);
    //设置打印浓度
    tsc.addDensity(DENSITY.DENSITY6);
    //设置标签间隙
    tsc.addGap(2);
    //添加图片
    tsc.addBitmap(0,0,BITMAP_MODE.OVERWRITE, 480, bitmap);
    //打印份数
    tsc.addPrint(count);
    //执行打印
    printer.print(tsc.getBytes());
}
```

## 4.2 小票指令（ESC）打印

```java
public static byte[] getESCChinese(Context context){
        EscCommand esc = new EscCommand();
        //初始化打印机
        esc.addInitializePrinter();
        //打印走纸多少个单位
        esc.addPrintAndFeedLines((byte) 3);
        // 设置打印居中
        esc.addSelectJustification(EscCommand.JUSTIFICATION.CENTER);
        // 设置为倍高倍宽
        esc.addSelectPrintModes(EscCommand.FONT.FONTA, EscCommand.ENABLE.OFF, EscCommand.ENABLE.ON, EscCommand.ENABLE.ON, EscCommand.ENABLE.OFF);
        // 打印文字
        esc.addText("票据测试\n");
        //打印并换行
        esc.addPrintAndLineFeed();
        // 取消倍高倍宽
        esc.addSelectPrintModes(EscCommand.FONT.FONTA, EscCommand.ENABLE.OFF, EscCommand.ENABLE.OFF, EscCommand.ENABLE.OFF, EscCommand.ENABLE.OFF);
        // 设置打印左对齐
        esc.addSelectJustification(EscCommand.JUSTIFICATION.LEFT);
        // 打印文字
        esc.addText("打印文字测试:\n");
        // 打印文字
        esc.addText("欢迎使用打印机!\n");
        esc.addPrintAndLineFeed();
        esc.addText("打印对齐方式测试:\n");
        // 设置打印左对齐
        esc.addSelectJustification(EscCommand.JUSTIFICATION.LEFT);
        esc.addText("居左");
        esc.addPrintAndLineFeed();
        // 设置打印居中对齐
        esc.addSelectJustification(EscCommand.JUSTIFICATION.CENTER);
        esc.addText("居中");
        esc.addPrintAndLineFeed();
        // 设置打印居右对齐
        esc.addSelectJustification(EscCommand.JUSTIFICATION.RIGHT);
        esc.addText("居右");
        esc.addPrintAndLineFeed();
        esc.addPrintAndLineFeed();
        // 设置打印左对齐
        esc.addSelectJustification(EscCommand.JUSTIFICATION.LEFT);
        esc.addText("打印Bitmap图测试:\n");
        Bitmap b = BitmapFactory.decodeResource(context.getResources(), R.mipmap.ic_priter);
        // 打印图片  光栅位图  384代表打印图片像素  0代表打印模式
        esc.addRastBitImage(b, 384, 0);
        esc.addPrintAndLineFeed();
        // 打印文字
        esc.addText("打印条码测试:\n");
        esc.addSelectPrintingPositionForHRICharacters(EscCommand.HRI_POSITION.BELOW);
        // 设置条码可识别字符位置在条码下方
        // 设置条码高度为60点
        esc.addSetBarcodeHeight((byte) 60);
        // 设置条码宽窄比为2
        esc.addSetBarcodeWidth((byte) 2);
        // 打印Code128码
        esc.addCODE128(esc.genCodeB("barcode128"));
        esc.addPrintAndLineFeed();
        /*
         * QRCode命令打印 此命令只在支持QRCode命令打印的机型才能使用。 在不支持二维码指令打印的机型上，则需要发送二维条码图片
         */
        esc.addText("打印二维码测试:\n");
        // 设置纠错等级
        esc.addSelectErrorCorrectionLevelForQRCode((byte) 0x31);
        // 设置qrcode模块大小
        esc.addSelectSizeOfModuleForQRCode((byte) 4);
        // 设置qrcode内容
        esc.addStoreQRCodeData("www.smarnet.cc");
        // 打印QRCode
        esc.addPrintQRCode();
        //打印并走纸换行
        esc.addPrintAndLineFeed();
        // 设置打印居中对齐
        esc.addSelectJustification(EscCommand.JUSTIFICATION.CENTER);
        //打印fontB文字字体
        esc.addSelectCharacterFont(EscCommand.FONT.FONTB);
        esc.addText("测试完成!\r\n");
        //打印并换行
        esc.addPrintAndLineFeed();
        //打印走纸n个单位
        esc.addPrintAndFeedLines((byte) 4);
        // 开钱箱
        esc.addGeneratePlus(LabelCommand.FOOT.F2, (byte) 255, (byte) 255);
        //开启切刀
        esc.addCutPaper();
        //添加缓冲区打印完成查询
        byte [] bytes={0x1D,0x72,0x01};
        //添加用户指令
        esc.addUserCommand(bytes);
        return esc.getCommand();
    }
```

测试内容打印结果如下

：![在这里插入图片描述](https://img-blog.csdnimg.cn/20190823101247931.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTIxODQ1Mzk=,size_16,color_FFFFFF,t_70)

# 5. 绘制打印内容图片

   SDK提供**BitmapCanvas**类用于快速绘制图片。其功能如下：

```java
public class BitmapCanvas {

    public BitmapCanvas(Context context, Bitmap bitmap, boolean antiAlias);

    /**
     * 绘制文本
     *
     * @param x        x位置
     * @param y        y位置
     * @param text     文本
     * @param textSize 字号
     * @return 添加后的Y位置
     */
    public int drawText(int x, int y, String text, int textSize);


    /**
     * 绘制文本
     *
     * @param x        x位置
     * @param y        y位置
     * @param text     文本
     * @param textSize 字号
     * @param typeface 字体
     * @return 添加后的Y位置
     */
    public int drawText(int x, int y, String text, int textSize, Typeface typeface);

    /**
     * 绘制文本
     *
     * @param x            x位置
     * @param y            y位置
     * @param text         文本
     * @param textSize     字号
     * @param typeface     字体
     * @param maxLineWidth 最大行宽，超出自动换行
     * @return 添加后的Y位置
     */
    public int drawText(int x, int y, String text, int textSize, Typeface typeface, int maxLineWidth);

    /**
     * 绘制文本
     *
     * @param x            x位置
     * @param y            y位置
     * @param text         文本内容，可以加\n实现换行
     * @param textSize     字号
     * @param typeface     字体
     * @param maxLineWidth 最大行宽，超出自动换行
     * @param letterSpace  字间距，单位是PX，默认是0
     * @param lineSpace    行间距，单位是PX，默认是0
     * @return 添加后的Y位置
     */
    public int drawText(int x, int y, String text, int textSize, Typeface typeface, int maxLineWidth, int letterSpace, int lineSpace);

    /**
     * 绘制图片
     *
     * @param x      x位置
     * @param y      y位置
     * @param bitmap 图片
     * @return 添加后的Y位置
     */
    public int drawImage(int x, int y, Bitmap bitmap);

    /**
     * 绘制线段
     *
     * @param x1        x1位置
     * @param y1        y1位置
     * @param x2        x2位置
     * @param y2        y2位置
     * @param lineWidth 线宽
     * @return 添加后的Y位置
     */
    public int drawLine(int x1, int y1, int x2, int y2, int lineWidth);

    /**
     * 绘制矩形
     *
     * @param x1        x1位置
     * @param y1        y1位置
     * @param x2        x2位置
     * @param y2        y2位置
     * @param lineWidth 线宽
     * @return 添加后的Y位置
     */
    public int drawRect(int x1, int y1, int x2, int y2, int lineWidth);

    /**
     * 获取绘制的图片
     *
     * @return 绘制的图片
     */
    public Bitmap getDrawBitmap();
}
```