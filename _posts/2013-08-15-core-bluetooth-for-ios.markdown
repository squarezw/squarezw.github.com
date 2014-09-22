---
layout: post
title: "iOS 上的蓝牙框架 - Core Bluetooth for iOS [译]"
date: 2013-08-15 09:52
comments: true
categories: CocoaFramework
tags: core bluetooth
---

所须环境: iOS 6 以上

原文: [Core Bluetooth for iOS 6](http://weblog.invasivecode.com/post/39707371281/core-bluetooth-for-ios-6-core-bluetooth-was)

Core Bluetooth 是在iOS5首次引入的，它允许iOS设备可以使用健康，运动，安全，自动化，娱乐，附近等外设数据。在iOS 6 中，这个API被扩展了，让iOS也能成为数据提供方，也就是`Server(Peripheral)`端，可能使它与其它 iOS 设备交互数据。

Core Bluetooth API 基于BLE4.0规范。这个框架涵盖了BLE标准的所有细节. 不过，仅仅只有新的iOS设备和MAC是兼容BLE标准的: iPhone 4S, iPhone5, Mac Mini, New iPad, MacBook Air, MacBook Pro. 并且 iOS 6 iPhone 模拟器也支持一样的标准.这对你在没有真机时，开发APP时是非常实用的。

## 相关的类

在CoreBluetooth框架中，有两个主要的角色：外设和中心（Peripheral and Central） ，整个框架都是围绕这两个主要角色设计的，它们之间有一系列的回调交换数据。
下图1展示了外设和中心（`Peripheral and Central`）的关系。

![Fig1](/assets/core_bluetooth_client_server_naming.png)

外设创建或提供一些数据，中心使用这些设备提供的数据。在iOS6之后，iOS 设备也可以即是外设，也可以是中心，但不能在同时间扮演两个角色。

这两个组件在CoreBluetooth框架中是分别用两个类来表示的，中央是`CBCentralManager`类，外设是`CBPeripheralManager`类。

在中心，一个 `CBPeripheral` 对象表示正在连接中的外设,同样在外设里，一个 `CBCentral` 表示正在连接中的中心.


你可以理解外设是一个广播数据的设备，它开始告诉外面的世界说它这儿有一些数据，并且能提供一些服务。另一边中心开始扫描外面有没有
自己所需要的服务，如果发现后，会和外设做连接请求，一旦连接确定后，两个设备就可以传输数据了。

除了中心与外设，我们还得考虑他们用于交互的数据结构，这些数据在Services(服务)中被结构化,每个服务由不同的Characteristics(特性)所组成。特性定义为一种属性类型，并且对应一个逻辑值(比如0x2A49)。

你可以在[developer bluetooth](http://developer.bluetooth.org)这里找到标准服务与特性的列表。

比如：

<!-- more -->

#Services:

    SpecificationName: Blood Pressure
    SpecificationType: org.bluetooth.service.blood_pressure
    AssignedNumber: 0x1810
    SpecificationLevel: Adopted

    #Characteristics:

    SpecificationName: Blood Pressure Feature
    SpecificationType: org.bluetooth.characteristic.blood_pressure_feature
    AssignedNumber: 0x2A49
    SpecificationLevel: Adopted

在中心里，服务由 `CBService` 类表示，每个服务由代表特性的 `CBCharacteristic`类所构成。

同样，在外设中服务与特性由 `CBMutableService` 与 `CBMutableCharacteristicclass` 类表示。

下图解释了他们之间的关系:

![Fig2](/assets/objects_involved_in_core_bluetooth.png)

`CBUUID` 和 `CBATTRequest` 是两个苹果提供给我们的帮助类，以便于开发者更简单地操作数据，稍后你将看到如何使用它们。


##使用

不幸的是，Apple提供的文档目前还不完整，你只有通过WWDC上两个关于 Core Bluetooth的视频和头文件，去理解这个框架是如何工作的。不过，因为我之前已经做过相关方面的事情，我决定和你分享这些内容，我希望下面的教程可以帮助到你。你也可以通过 http://training.invasivecode.com 查看我们的培训课程.


#### 创建外设 (Peripheral)

为了可以创建一个完整的例子，你需要两台iOS设备，我将向你展示如何通过蓝牙连接这两个设备，并交换数据。记住先检查一下你的设备是不是被BLE所支持的。

开始创建一个外设需要下面几步：

1.创建并且开始Peripheral Manager

2.设置并且发布它的服务。

3.广播这个服务。

4.和中心连接。

用Single-View Application模板创建一个新的Xcode工程。命名为BlueServer （使用ARC）。工程创建完成后，添加CoreBluetooth.framework 框架。然后打开ViewController.h文件，并且添加以下代码：

    #import <CoreBluetooth/CoreBluetooth.h>

使view controller 遵循 `CBPeripheralManagerDelegate` 协议，然后添加这个属性：

    @property (nonatomic, strong) CBPeripheralManager *manager;

在ViewController.m中，添加以下代码到viewDidLoad方法中：

    self.manager = [[CBPeripheralManager alloc] initWithDelegate:self queue:nil];

这行代码初始化了一个 Peripheral Manager (计划中的第一项). 第一个参数是设置delegate(这里的例子就是view controller)。第二参数(队列)设置为了nil,因为Peripheral Manager 将运行在主线程中。如果你想用同步的线程做更复杂的事情，你需要单独创建一个队列并把它放在这个参数中。

一旦Peripheral Manager被初始化后，我们需要及时检查正在运行的App设备状态，是不是符合BLE标准的。所以你要实现下面的这个代理方法（如果设备不支持BLE,你可以友好地提醒用户。你还可以通过拿到的状态值做更多事情）。

```
- (void)peripheralManagerDidUpdateState:(CBPeripheralManager *)peripheral {
    switch (peripheral.state) {
        case CBPeripheralManagerStatePoweredOn:
            [self setupService];
            break;
        default:
            NSLog(@"Peripheral Manager did change state");
            break;
    }
}
```

在这里，我检查了外设的状态，如果它的状态是`CBPeripheralManagerStatePoweredOn`，那这个设备是支持BLE并可以继续执行。

外设的状态包括有下面这些

```
typedef enum {
   CBPeripheralManagerStateUnknown = 0,
   CBPeripheralManagerStateResetting,
   CBPeripheralManagerStateUnsupported,
   CBPeripheralManagerStateUnauthorized,
   CBPeripheralManagerStatePoweredOff,
   CBPeripheralManagerStatePoweredOn,
} CBPeripheralManagerState;
```

## 服务与特性 (Service & Characteristic)

setupService 是一个辅助方法，我们让它去准备服务和特性，对于这个例子，我们仅仅需要一个服务和一个特性。

每一个服务和特性必要有一个UUID来标识，UUID是一个16位或128位的值。如果你创建的是一个 client-server(中央-外设)应用，那么你需要创建属于你自己的128位UUID，你必须确保它不能和其他已经存在的服务冲突，如果你要创建一个新的设备，你需要去符合标准委员会的UUID。

如果你创建的是你自己的client-server(正如我们现在做的)，我建议你在Terminal下用 uuidgen 命令创建128位的UUID. 所以打开 Terminal并创建两个（一个为服务，一个为特性）.之后，你需要将他们添加到中心和外设应用。这里我们先添加下面几行在 view controller中。

    static NSString * const kServiceUUID = @"6BC6543C-2398-4E4A-AF28-E4E0BF58D6BC";
    static NSString * const kCharacteristicUUID = @"9D69C18C-186C-45EA-A7DA-6ED7500E9C97";

注意：这里的UUID每个人生成的都不一样，最好是你自己生成

这里是 setupService 的实现方法:

```
- (void)setupService {
    // Creates the characteristic UUID
    CBUUID *characteristicUUID = [CBUUID UUIDWithString:kCharacteristicUUID];

    // Creates the characteristic
    self.customCharacteristic = [[CBMutableCharacteristic alloc] initWithType:characteristicUUID properties:CBCharacteristicPropertyNotify value:nil permissions:CBAttributePermissionsReadable];

    // Creates the service UUID
    CBUUID *serviceUUID = [CBUUID UUIDWithString:kServiceUUID];

    // Creates the service and adds the characteristic to it
    self.customService = [[CBMutableService alloc] initWithType:serviceUUID primary:YES];

    // Sets the characteristics for this service
    [self.customService setCharacteristics:@[self.customCharacteristic]];

    // Publishes the service
    [self.peripheralManager addService:self.customService];
} 
```

首先，我使用`+UUIDWithString:`方法创建了一个 UUID 对象，之后我用这个 UUID对象创建了特性。注意，我在初始化时，第三个参数传的是nil (那个value)，之所以这样做，是因为我告诉 Core Bluetooth我将稍候添加这个特性值，当你需要动态创建数据时，经常这么做。如果你已经有一个静态的值，你可以直接传它。

在这个方法中，第一个参数是先创建好的UUID,第二个参数(那个 properties)确定你将如何使用这个特性值，下面是这些可能的值：

    CBCharacteristicPropertyBroadcast: 允许一个广播特性值,用于描述特性配置，不允许本地特性
    CBCharacteristicPropertyRead: 允许读一个特性值
    CBCharacteristicPropertyWriteWithoutResponse: 允许写一个特性值，没有反馈
    CBCharacteristicPropertyWrite: 允许写一个特性值
    CBCharacteristicPropertyNotify: 允许通知一个特性值，没有反馈
    CBCharacteristicPropertyIndicate: 允许标识一个特性值
    CBCharacteristicPropertyAuthenticatedSignedWrites: 允许签名一个可写的特性值
    CBCharacteristicPropertyExtendedProperties: 如果设置后，附加特性属性为一个扩展的属性说明，不允许本地特性
    CBCharacteristicPropertyNotifyEncryptionRequired: 如果设置后，仅允许信任的设备可以打开通知特性值
    CBCharacteristicPropertyIndicateEncryptionRequired: 如果设置后，仅允许信任的设备可以打开标识特性值

最后一个参数是属性的读、写、加密权限，有以下几种：

    CBAttributePermissionsReadable
    CBAttributePermissionsWriteable
    CBAttributePermissionsReadEncryptionRequired
    CBAttributePermissionsWriteEncryptionRequired

创建特性后，我同样通过`+UUIDWithString:`方法创建 UUID，然后通过它创建了服务。 最后我为服务设置对应的这个特性。记住，每个服务可以包括多个特性，正如下面的图3

![Fig3](/assets/services_and_characteristics.png)

所以我们需要通过一个特性数组来添加到服务中，在这个例子里，这个数组对象只有一个特性。

最后一行的代码是将服务添加到 Peripheral Manager中，用于发布这个服务。一旦这样做之后，Peripheral Manager 将会通知它的 delegate调用`peripheralManager:didAddService:error:`方法。这里如果没有错误，你可以开始广播这个服务。

```
- (void)peripheralManager:(CBPeripheralManager *)peripheral didAddService:(CBService *)service error:(NSError *)error {
    if (error == nil) {
        // Starts advertising the service
        [self.peripheralManager startAdvertising:@{ CBAdvertisementDataLocalNameKey : @"ICServer", CBAdvertisementDataServiceUUIDsKey : @[[CBUUID UUIDWithString:kServiceUUID]] }];
    }
}
```

当Peripheral Manager开始广播这个服务时，delegate 会接收到 `peripheralManagerDidStartAdvertising:error:` 消息。当中心订阅
了这个服务时，它的delegate会收到 `peripheralManager:central:didSubscribeToCharacteristic:`消息，这儿你可以生成动态数据给中心。

现在，发送数据给中心你需要预先写一些数据的代码，然后发送`updateValue:forCharacteristic:onSubscribedCentrals:`到外设。


## 创建一个中心 (Central)

现在，我们已经有了一个外设，让我们创建中心(client)。记住，中心是用来处理外设提供的数据的。如上面的图2所示，这里的中心被 `CBCentralManager` 对象表示。

让我们创建一个名字为 BlueClient 的 Xcode 项目，使用ARC,并添加 CoreBluetooth.framework ，在 view controller 头添加

    #import <CoreBluetooth/CoreBluetooth.h>

在中心，你必须遵循两个协议: CBCentralManagerDelegate 和 CBPeripheralDelegate

    @interface ViewController : UIViewController <CBCentralManagerDelegate, CBPeripheralDelegate>

并添加两个属性:

    @property (nonatomic, strong) CBCentralManager *manager;
    @property (nonatomic, strong) NSMutableData *data;

现在正如我们之前为外设创建的做法一样，我们创建中心对象:
    self.manager = [[CBCentralManager alloc] initWithDelegate:self queue:nil];

同样，这里的第一个参数表示 CBCentralManager delegate (这里是 view controller). 第二个参数和之前一样也表示调度队列，如果设置为空，他会使用主队列。

一旦 Central Manager 初始化后，我们同样也要检查它的状态，是不是被 BLE 所支持的APP，实现下面的delegate 方法:

```
- (void)centralManagerDidUpdateState:(CBCentralManager *)central {
    switch (central.state) {
        case CBCentralManagerStatePoweredOn:
            // Scans for any peripheral
            [self.manager scanForPeripheralsWithServices:@[ [CBUUID UUIDWithString:kServiceUUID] ] options:@{CBCentralManagerScanOptionAllowDuplicatesKey : @YES }];
            break;
        default:
            NSLog(@"Central Manager did change state");
            break;
    }
}
```

这个`scanForPeripheralsWithServices:options:` 方法用于告诉 Central Manager 开始查看特别的服务，如果你第一个参数用的是nil，这个Central Manager 开始查看所有服务。

这个 kServiceUUID 和创建外设中的 ServiceUUID 一样。所以我们再次添加下面2行代码在你的实现类中。

    static NSString * const kServiceUUID = @"6BC6543C-2398-4E4A-AF28-E4E0BF58D6BC";
    static NSString * const kCharacteristicUUID = @"9D69C18C-186C-45EA-A7DA-6ED7500E9C97";

一旦一个外设在扫描时被发现后，中心 delegate 会收到下面的回调：

    - (void)centralManager:(CBCentralManager *)central didDiscoverPeripheral:(CBPeripheral *)peripheral advertisementData:(NSDictionary *)advertisementData RSSI:(NSNumber *)RSSI 

这个调用通知Central Manager delegate（在这个例子中就是view controller），一个附带着广播数据和信号质量(RSSI-Received Signal Strength Indicator)的周边被发现。这是一个很酷的参数，知道了信号质量，你可以用它去估计中心与外设的距离。

任何广播或扫描的响应数据保存在advertisementData 中，可以通过CBAdvertisementData key来访问它。现在，你可以停止扫描，去连接外设了：

```
- (void)centralManager:(CBCentralManager *)central didDiscoverPeripheral:(CBPeripheral *)peripheral advertisementData:(NSDictionary *)advertisementData RSSI:(NSNumber *)RSSI {
    // Stops scanning for peripheral
    [self.manager stopScan];

    if (self.peripheral != peripheral) {
        self.peripheral = peripheral;
        NSLog(@"Connecting to peripheral %@", peripheral);
        // Connects to the discovered peripheral
        [self.manager connectPeripheral:peripheral options:nil];
    }
}
```

options 参数是一个可选的字典(NSDictionary)，如果需要，可以用以下的键(keys), 它们的值始终是一个boolean。

    CBConnectPeripheralOptionNotifyOnConnectionKey: 这是一个NSNumber(Boolean)，表示系统会为获得的外设显示一个提示，当成功连接后这个应用被挂起，这对于没有运行在中心后台模式并不显示他们自己的提示时是有用的。如果有更多的外设连接后都会发送通知，如果附近的外设运行在前台则会收到这个提示。
    CBConnectPeripheralOptionNotifyOnDisconnectionKey:  这是一个NSNumber(Boolean), 表示系统会为获得的外设显示一个关闭提示，如果这个时候关闭了连接，这个应用会挂起。
    CBConnectPeripheralOptionNotifyOnNotificationKey: 这是一个NSNumber(Boolean)，表示系统会为获得的外设收到通知后显示一个提示，这个时候应用是被挂起的。

基于连接的结果，delegate会接收 

```
- centralManager:didFailToConnectPeripheral:error:
```

或者
```
- centralManager:didConnectPeripheral:
```
中的一个。如果成功了，你可以询问正在广播服务的那个外设。因此，在didConnectPeripheral 回调中，你可以写以下代码：

```
- (void)centralManager:(CBCentralManager *)central didConnectPeripheral:(CBPeripheral *)peripheral {
    // Clears the data that we may already have
    [self.data setLength:0];
    // Sets the peripheral delegate
    [self.peripheral setDelegate:self];
    // Asks the peripheral to discover the service
    [self.peripheral discoverServices:@[ [CBUUID UUIDWithString:kServiceUUID] ]];
}
```

现在，外设开始用一串回调通知它的delegate。在上面一个方法中，我请求外设去寻找服务，外设代理收到 -peripheral:didDiscoverServices:
如果没有错误，外设可以去查找服务所提供特性，你可以这样做。

```
- (void)peripheral:(CBPeripheral *)aPeripheral didDiscoverServices:(NSError *)error {
    if (error) {
        NSLog(@"Error discovering service: %@", [error localizedDescription]);
        [self cleanup];
        return;
    }

    for (CBService *service in aPeripheral.services) {
        NSLog(@"Service found with UUID: %@", service.UUID);

        // Discovers the characteristics for a given service
        if ([service.UUID isEqual:[CBUUID UUIDWithString:kServiceUUID]]) {
            [self.peripheral discoverCharacteristics:@[[CBUUID UUIDWithString:kCharacteristicUUID]] forService:service];
        }
    }
}
```

现在，如果一个特性被发现，外设delegate 又会接收
 
    peripheral:didDiscoverCharacteristicsForService:error:

```
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverCharacteristicsForService:(CBService *)service error:(NSError *)error {
    if (error) {
        NSLog(@"Error discovering characteristic: %@", [error localizedDescription]);
        [self cleanup];
        return;
    }
    if ([service.UUID isEqual:[CBUUID UUIDWithString:kServiceUUID]]) {
        for (CBCharacteristic *characteristic in service.characteristics) {
            if ([characteristic.UUID isEqual:[CBUUID UUIDWithString:kCharacteristicUUID]]) {
                [peripheral setNotifyValue:YES forCharacteristic:characteristic];
            }
        }
    }
}
```
一旦特征的值用`setNotifyValue:forCharacteristic:` 更新后，外设就会通知它的delegate。

外设的 delegate 就会接收到 
    peripheral:didUpdateNotificationStateForCharacteristic:error:

这里，你可以用 `readValueForCharacteristic:` 读到新的值

```
- (void)peripheral:(CBPeripheral *)peripheral didUpdateNotificationStateForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error {
    if (error) {
        NSLog(@"Error changing notification state: %@", error.localizedDescription);
    }

    // Exits if it's not the transfer characteristic
    if (![characteristic.UUID isEqual:[CBUUID UUIDWithString:kCharacteristicUUID]]) {
        return;
    }

    // Notification has started
    if (characteristic.isNotifying) {
        NSLog(@"Notification began on %@", characteristic);
        [peripheral readValueForCharacteristic:characteristic];
    } else { // Notification has stopped
        // so disconnect from the peripheral
        NSLog(@"Notification stopped on %@.  Disconnecting", characteristic);
        [self.manager cancelPeripheralConnection:self.peripheral];
    }
}
```
当外设发送新的值时，外设的 delegate 会收到 `peripheral:didUpdateValueForCharacteristic:error:`，这个方法的第二个参数包含特性，你可以用 `-value` 属性来读它，这是一个包含了特性值的NSData。

这个时候，你可以为其它数据断开或等待。

##总结

我为你展示了如何使用 Core Bluetooth 框架的基本示例，我希望通过这个教程，加上WWDC视频，有用的一些文档能帮助你创建一个 BLE 项目，同时你也可以去参考一些文档示例，那你会发现我这教程中所有的 delegate 方法。

