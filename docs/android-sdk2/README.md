
# 目录

 [TOC]
# 1. SDK介绍

此文档介绍佳博打印机SDK使用方式。SDK集成了打印机的多种连接方式，开发者可自己选择不同的方式连接打印机，也可自己实现。在SDK中APP与打印机连接的方式有5种，分别为：

1. 无线局域网接入（TCP）
2. Android USB接入
3. Android USB接入（配件模式）
4. 串口接入
5. 蓝牙接入

开发者需要根据打印机所支持的方式选择合适的接入方式。

SDK2是多连接版本（支持同时连接多台打印机），其大部分接口与单连接版本相同，如需使用原有单连接版本请查看传送门：[单连接版本SDK](../sdk)

# 2. 打印机指令集

上位机与打印机通信的指令集有多种，开发者可以自己连接打印机后通过指令与打印机通信，此SDK已集成指令集接口。

打印机可支持如下的一种或多种指令集，根据实际情况选择使用。各指令集的功能如下：

| 指令集           | 介绍                              |
| ---------------- | --------------------------------- |
| Esc或EscCommand  | 票据打印指令集                    |
| EscForDotPrinter | 票据指令集（GP-76系列针式打印机） |
| TscCommand       | 标签打印指令集                    |
| CpclCommand      | 面单打印指令集                    |
| ZplCommand       | 斑马指令集                        |

- 各指令的详细说明请查看佳博打印机编程手册
- Esc是EscCommand的重构版本，指令内容一样
- EscForDotPrinter是Esc的子类，Esc中的部分指令在GP-76系列针式打印机上无效，故新增EscForDotPrinter类把这些无效的指令添加了@Deprecated标志。
- 指令在不同型号的打印机上支持程度可能有区别，部分指令可能无效
- 

# 3. 快速接入

## 3.1 接入方式

在项目根目的的build.gradle文件中添加：

```gradle
repositories {
	maven {
		url  "http://118.31.6.84:8081/repository/maven-public/"
		//如使用gradle7.0及以上版本，请设置下面的值为true
		allowInsecureProtocol true
	}
}
```

在需要使用SDK的模块的build.gradle中添加：

```gradle
dependencies {
    implementation 'com.gainscha:sdk2:1.0.3'
}
```

这里介绍如何使用SDK集成的连接接口连接并控制打印机

## 3.2 添加相关权限

另外，需要动态申请敏感权限，在APP启动时要动态申请两个权限

```xml
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

开发者需自行编写代码申请这两个权限

## 3.3 相关类文件介绍

| 类名 | 说明 |
|--|--|
| Printer | 获取可用的打印机设备列表，连接打印机 |
|ConnectionListener|打印机连接状态监听|

## 3.4 监听打印机连接状态

```java
ConnectionListener listener = new ConnectionListener(){
    @Override
    public void onPrinterConnected(String printerId) {
		//连接成功
    }

    @Override
    public void onPrinterConnectFail(String printerId) {
		//连接失败
    }

    @Override
    public void onPrinterDisconnect(String printerId) {
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

SDK支持多种方式连接到打印机，可以通过调用Printer类中的方法连接打印机。SDK支持同时new多个Printer对象去连接不同的打印机，通过printerId区分不同的打印机。

```java
//创建printer对象，可传入printerId，不传printerId时会生成默认的
Printer printer=new Printer();
        Printer printer=new Printer("Printer_123");
//使用蓝牙连接到打印机
        printer.connectByBlueTooth();
//使用串口连接到打印机
        printer.connectBySerial();
//使用网络连接到打印机
        printer.connectByTcp();
//使用USB连接到打印机
        printer.connectByUsb();
//使用USB连接到打印机（配件模式）
        printer.connectByUsbAccessory();
```

Printer类提供如下的静态方法，方便多打印机管理

```java
/**
 * 设置是否打印日志，日志保存文件，需确保应用具有文件读写权限
 * 日志文件最大20M,超出大小后删除旧文件重新写入
 *
 * @param enable     是否打印日志
 * @param outputFile 日志输出文件, 值为null时不保存到文件
 */
public static void setLogEnable(boolean enable,File outputFile);
/**
 * 获取已连接的打印机列表
 */
public static Map<String, Printer> getConnectedPrinters();
/**
 * 生成随机ID
 */
private static String randomPrinterId();
/**
 * 获取已连接的打印机实例
 */
public static Printer getPrinter(String printerId);
/**
 * 判断打印机是否已连接
 *
 * @param printerId 打印机ID
 * @return 是否已连接
 */
public static boolean isConnected(String printerId);
/**
 * 断开打印机连接
 */
public static void disconnect(String printerId);
/**
 * 添加打印机连接状态监听回调
 *
 * @param connectionListener 监听回调
 */
public static void addConnectionListener(ConnectionListener connectionListener);
/**
 * 解除打印机连接状态监听回调
 *
 * @param connectionListener 监听回调
 */
public static void removeConnectionListener(ConnectionListener connectionListener);
```

## 3.5 打印内容

连接到打印机后，可以调用如下方法发送数据到打印机，实现打印。

```java
Printer printer = Printer.getPrinter(printerId);
if(printer == null && !printer.isConnected()){
    //打印机未连接
    return;
}
//发送数据到打印机，阻塞发送
//使用TscCommand、EscCommand、CpclCommand、ZplCommand类生成打印数据
printer.print(byte[] data);

//异步发送数据到打印机，并读取打印机的响应内容（超时时间内首次接收到的数据）
printer.print(byte[] data, int responseTimeOut, PrinterResponse<byte[]> onResponse);

//异步发送数据到打印机，并读取打印机的响应内容
//responseUntilTimeout是否读取数据直到超时，当为true时会返回超时时间内读取的全部数据
printer.print(byte[] data, int responseTimeOut, boolean responseUntilTimeout, PrinterResponse<byte[]> onResponse);

//异步打印图片
printer.print(Bitmap bitmap, int count, PrinterConfig config);
```


# 4. 例程

## 4.1 打印图片

~~~java
public void print(Bitmap bitmap, int count){
    PrinterConfig config = new PrinterConfig();
    //指令
    config.setInstruction(Instruction.ESC);
    //浓度（1-15）
    config.setDensity(2);
    //间隙（毫米）
    config.setGap(0);
    //打印速度（1-15）
    config.setSpeed(4);
    //撕离
    config.setTearMode(TearMode.TEAR);
    //纸张类型
    config.setPaperType(PaperType.PAPER_TYPE_CONTINUOUS);
    //是否抖动处理图片
    config.setBitmapShake(false);
    //是否切分图片（长图拆分为多个标签打印）
    config.setSlice(false);
    //打印机分辨率
    config.setDpi(Dpi.DPI_203);
    //打印标签宽高（毫米）
    config.setSize(printBitmap.getWidth() / 8, printBitmap.getHeight() / 8);
    //打印
    printer.print(printBitmap, 1, config);
}
~~~

## 4.2 打印图文

使用原始指令具有更加强大功能，可以设置更多的参数、添加更多的打印内容。如打印文本、线段、条码等。其缺点就是每种指令集都具有不同的实现方式，在需要切换指令集的情况下比较麻烦，需要编码多套代码。

### 4.2.1 标签指令（TSPL）打印

~~~java
public void print(Bitmap bitmap, int count){
    TscCommand tsc = new TscCommand();
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
    printer.print(tsc.getCommand());
}
~~~

### 4.2.2 小票指令（ESC）打印

~~~java
    EscCommand cmd = new EscCommand();
    //初始化
    cmd.addInitializePrinter();
    //居中对齐
    cmd.addSelectJustification(EscCommand.JUSTIFICATION.CENTER);
    cmd.addSetPrintingAreaWidth((short) 480);
    //切刀
    cmd.addCutPaper();
    //打印文本
    cmd.addText("打印文本");
    //打印条码
    cmd.addCODE128("01234567");
    //打印图片
    cmd.addBitmap(bitmap);
    //执行打印
    printer.print(cmd.getCommand());
~~~

## 4.2.3 小票指令（ESC）打印

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

# 5. 搜索打印机

   SDK提供了搜索打印机类(**PrinterFinder.java**)，实际开发中可根据需要选择是否使用。提供如下的功能：

```java
    /**
    * 获取所有连接方式的设备列表
    *
    * @param searchPrinterResultListener 查找到新设备回调
    */
    public void searchPrinters(Context context, SearchPrinterResultListener searchPrinterResultListener);
    /**
     * 获取可连接的设备列表
     *
     * @param connectType    连接方式
     * @param listener       查找到新设备回调
     */
    public void searchPrinters(Context context, ConnectType connectType, final SearchPrinterResultListener listener);
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
         * @param bluetoothDevice 蓝牙设备
         */
        void onSearchBluetoothPrinter(BluetoothDevice bluetoothDevice, int rssi);

        /**
         * 搜索到USB设备
         *
         * @param usbDevice usb设备
         * @param rssi      信号强度
         */
        void onSearchUsbPrinter(UsbDevice usbDevice);

        /**
         * 搜索到USB设备（配件模式）
         *
         * @param usbAccessory usb配件
         */
        void onSearchUsbPrinter(UsbAccessory usbAccessory);

        /**
         * 搜索网络设备
         *
         * @param mac     MAC地址
         * @param ip      IP地址
         * @param gateway 网关
         * @param netmask 掩码
         * @param port    端口
         */
        void onSearchNetworkPrinter(String mac, String ip, String gateway, String netmask, int port);

        /**
         * 搜索到串口设备
         *
         * @param serialPortPath 串口文件地址
         */
        void onSearchSerialPortPrinter(String serialPortPath);

        /**
         * 搜索完成
         */
        void onSearchCompleted();
    }
```

# 6. 绘制打印内容图片

   使用TscCommand、EscCommand等指令生成类来构建打印内容具有一定的限制性，如，不能自定义字体（只能使用打印机内置的一些字体）、打印的内容不能在上位机预览（只能通过打印后查看打印结果）等。

   在Android中可以通过Canvas自定义绘制任意内容，生产bitmap后执行打印。使用这种方式生成的打印内容可以实时预览。但是发送给打印机数据量较大（使用蓝牙传输时传输速度差距大）.

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
     * 根据指定的宽度拆分文本为多行文本
     */
    private List<String> splitTextForWidth(String text, int maxLineWidth, Paint paint);

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