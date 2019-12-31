## 设备管理

设备管理相关的所有功能对应`WiserSmartDevice`类，需要使用设备Id进行初始化。错误的设备Id可能会导致初始化失败，返回`nil`。

### 更新设备信息

#### 更新单个设备信息

Objc:

```objc
- (void)updateDeviceInfo {
	// self.device = [WiserSmartDevice deviceWithDeviceId:@"your_device_id"];

	__weak typeof(self) weakSelf = self;
	[self.device syncWithCloud:^{
		NSLog(@"syncWithCloud success");
		NSLog(@"deviceModel: %@", weakSelf.device.deviceModel);
	} failure:^(NSError *error) {
		NSLog(@"syncWithCloud failure");
	}];
}
```

Swift:

```swift
func updateDeviceInfo() {
    device?.sync(cloud: {
        print("syncWithCloud success")
    }, failure: { (error) in
        if let e = error {
            print("syncWithCloud failure: \(e)")
        }
    })
}
```



###设备控制

设备控制目前分为**标准设备控制**和**自定义设备控制**



#### 标准设备控制 (Beta)

##### 标准设备功能集

什么是标准设备功能集？

不同品类的产品，功能点都不一样。例如照明品类中，以灯为例，有开关、调色等功能，但对于电工品类，以插座为例，就没有 “调色” 功能

但对于某个大的品类而言，基本的通用功能都是一致的。比如，所有的照明品类，都有开关这个功能

统一同类产品的功能集定义，制定一套功能指令集规则，这就是标准功能集

> 因为需要兼容繁多的多样化产品功能，所以目前标准设备控制功能处于阶段性开放产品适配功能
> 若您要使用该功能，可以联系施耐德或咨询相关的对接负责人



##### 设备否支持标准化

`WiserSmartDeviceModel` 类的 `standard` 属性（ `BOOL` 类型）定义了当前设备是否支持标准化控制

`dpCodes` 属性定义了当前设备的状态，称作标准数据点(标准 dp code)

`dpCodes` 字典里的每个 `key` 对应一个功能点的 `dpCode`，`value` 对应一个功能点的 `dpValue `，`dpValue` 为该功能点的值

具体品类的设备标准 dpCode 功能集可以参照对应的[文档](./StandardDpCode.md)



#####设备操作控制

```objective-c
[self.device publishDpWithCommands:dpCodesCommand success:^{
    NSLog(@"publishDpWithCommands success");
} failure:^(NSError *error) {
    NSLog(@"publishDpWithCommands failure: %@", error);
}];
```

【指令格式】

```json
// 以灯为例
// 每种产品都有标准的一套规则
// 作用: 开关打开 
{"switch_led" : true}

// 字符串型功能点示例 作用: 设置彩光颜色
{"colour_data" : "009003e803e8"}

// 枚举型功能点示例 作用: 设置模式为 "white" 白光模式
{"work_mode" : "white"}

// 设置数值型功能点示例 作用: 设置亮度为100
{"bright_value" : 100}

// 多个功能合并发送
{"work_mode" : "colour"}
{"colour_data" : "009003e803e8"}
```

##### 设备状态更新

实现 `WiserSmartDeviceDelegate`代理协议后，可以在设备状态更变的回调中进行处理，刷新APP设备控制面板的UI。

Objc:

```objective-c
- (void)initDevice {
    self.device = [WiserSmartDevice deviceWithDeviceId:@"your_device_id"];
    self.device.delegate = self;
}

#pragma mark - WiserSmartDeviceDelegate

- (void)device:(WiserSmartDevice *)device dpCommandsUpdate:(NSDictionary *)dpCodes {
    NSLog(@"dpCommandsUpdate: %@", dpCodes);
    // TODO: 刷新界面UI
}

- (void)deviceInfoUpdate:(WiserSmartDevice *)device {
    //当前设备信息更新 比如 设备名、设备在线状态等
}

- (void)deviceRemoved:(WiserSmartDevice *)device {
    //当前设备被移除
}
```

Swift:

```objective-c
func initDevice() {
    device = WiserSmartDevice(deviceId: "your_device_id")
    device?.delegate = self
}

// MARK: - WiserSmartDeviceDelegate

func device(_ device: WiserSmartDevice!, dpCommandsUpdate dpCodes: [AnyHashable : Any]!) {
    print("dpCommandsUpdate: \(dpCodes)")
    // TODO: 刷新界面UI
}

func deviceInfoUpdate(_ device: WiserSmartDevice!) {
    //当前设备信息更新 比如 设备名、设备在线状态等
}

func deviceRemoved(_ device: WiserSmartDevice!) {
    //当前设备被移除
}
```

#### 自定义设备控制

##### 设备功能点

`WiserSmartDeviceModel`类的`dps`属性（`NSDictionary`类型）定义了当前设备的状态，称作数据点（DP点）或功能点。

`dps`字典里的每个`key`对应一个功能点的`dpId`，`value`对应一个功能点的`dpValue `，`dpValue`为该功能点的值。<br />


发送控制指令按照以下格式：

`{"<dpId>":"<dpValue>"}`

根据后台该产品的功能点定义，示例代码如下:

Objc:

```objc
- (void)publishDps {
    // self.device = [WiserSmartDevice deviceWithDeviceId:@"your_device_id"];

    NSDictionary *dps;
	//设置dpId为1的布尔型功能点示例 作用:开关打开
	dps = @{@"1": @(YES)};

	//设置dpId为4的字符串型功能点示例 作用:设置RGB颜色为ff5500
	dps = @{@"4": @"ff5500"};

	//设置dpId为5的枚举型功能点示例 作用:设置档位为2档
	dps = @{@"5": @"2"};

	//设置dpId为6的数值型功能点示例 作用:设置温度为20°
	dps = @{@"6": @(20)};

	//设置dpId为15的透传型(byte数组)功能点示例 作用:透传红外数据为1122
	dps = @{@"15": @"1122"};

	//多个功能合并发送
	dps = @{@"1": @(YES), @"4": @(ff5500)};

	[self.device publishDps:dps success:^{
		NSLog(@"publishDps success");

		//下发成功，状态上报通过 deviceDpsUpdate方法 回调

	} failure:^(NSError *error) {
		NSLog(@"publishDps failure: %@", error);
	}];

}
```
Swift:

```swift
func publishDps() {
    var dps = [String : Any]()
 
    // dp 可参考具体产品定义
    device?.publishDps(dps, success: {
 	    print("publishDps success")
        
        //下发成功，状态上报通过 deviceDpsUpdate方法 回调
    }, failure: { (error) in
        if let e = error {
            print("publishDps failure: \(e)")
        }
    })
}
```

##### 注意事项：

- 控制命令的发送需要特别注意数据类型。<br />
  比如功能点的数据类型是数值型（value），那控制命令发送的应该是 `@{@"2": @(25)}`  而不是  `@{@"2": @"25"}`<br />
- 透传类型传输的byte数组是字符串格式、字母需小写并且必须是偶数位。<br />
  比如正确的格式是: `@{@"1": @"011f"}` 而不是 `@{@"1": @"11f"}`

##### 设备控制

设备控制支持三种通道控制，局域网控制，云端控制，和自动方式（如果局域网在线，先走局域网控制，局域网不在线，走云端控制）

局域网控制：

```
	[self.device publishDps:dps mode:TYDevicePublishModeLocal success:^{
		NSLog(@"publishDps success");
	} failure:^(NSError *error) {
		NSLog(@"publishDps failure: %@", error);
	}];
```

云端控制：

```
	[self.device publishDps:dps mode:TYDevicePublishModeInternet success:^{
		NSLog(@"publishDps success");
	} failure:^(NSError *error) {
		NSLog(@"publishDps failure: %@", error);
	}];
```

自动控制：

```
	[self.device publishDps:dps mode:TYDevicePublishModeAuto success:^{
		NSLog(@"publishDps success");
	} failure:^(NSError *error) {
		NSLog(@"publishDps failure: %@", error);
	}];
```


##### 设备状态更新

实现`WiserSmartDeviceDelegate`代理协议后，可以在设备状态更变的回调中进行处理，刷新APP设备控制面板的UI。

Objc:

```objc
- (void)initDevice {
	self.device = [WiserSmartDevice deviceWithDeviceId:@"your_device_id"];
	self.device.delegate = self;
}

#pragma mark - WiserSmartDeviceDelegate

- (void)device:(WiserSmartDevice *)device dpsUpdate:(NSDictionary *)dps {
	NSLog(@"deviceDpsUpdate: %@", dps);
	// TODO: 刷新界面UI
}

- (void)deviceInfoUpdate:(WiserSmartDevice *)device {
	//当前设备信息更新 比如 设备名、设备在线状态等
}

- (void)deviceRemoved:(WiserSmartDevice *)device {
	//当前设备被移除
}

```

Swift:

```swift
func initDevice() {
    device = WiserSmartDevice(deviceId: "your_device_id")
    device?.delegate = self
}

// MARK: - WiserSmartDeviceDelegate

func device(_ device: WiserSmartDevice!, dpsUpdate dps: [AnyHashable : Any]!) {
    print("deviceDpsUpdate: \(dps)")
    // TODO: 刷新界面UI
}

func deviceInfoUpdate(_ device: WiserSmartDevice!) {
    //当前设备信息更新 比如 设备名、设备在线状态等
}

func deviceRemoved(_ device: WiserSmartDevice!) {
    //当前设备被移除
}
```



### 修改设备名称

Objc:

```objc
- (void)modifyDeviceName:(NSString *)mame {
	// self.device = [WiserSmartDevice deviceWithDeviceId:@"your_device_id"];

	[self.device updateName:name success:^{
		NSLog(@"updateName success");
	} failure:^(NSError *error) {
		NSLog(@"updateName failure: %@", error);
	}];
}
```

Swift:

```swift
func modifyDeviceName(_ name: String) {
    device?.updateName(name, success: {
        print("updateName success")
    }, failure: { (error) in
        if let e = error {
            print("updateName failure: \(e)")
        }
    })
}
```



### 移除设备

设备被移除后，会重新进入待配网状态（快连模式）。

Objc:

```objc
- (void)removeDevice {
	// self.device = [WiserSmartDevice deviceWithDeviceId:@"your_device_id"];

	[self.device remove:^{
		NSLog(@"remove success");
	} failure:^(NSError *error) {
		NSLog(@"remove failure: %@", error);
	}];
}
```

Swift:

```swift
func removeDevice() {
    device?.remove({
        print("remove success")
    }, failure: { (error) in
        if let e = error {
            print("remove failure: \(e)")
        }
    })
}
```



### 获取设备的wifi信号强度

Objc:

```objc
- (void)getWifiSignalStrength {
	// self.device = [WiserSmartDevice deviceWithDeviceId:@"your_device_id"];
    // self.device.delegate = self;

	[self.device getWifiSignalStrengthWithSuccess:^{
		NSLog(@"get wifi signal strength success");
	} failure:^(NSError *error) {
		NSLog(@"get wifi signal strength failure: %@", error);
	}];
}

#pragma mark - WiserSmartDeviceDelegate

- (void)device:(WiserSmartDevice *)device signal:(NSString *)signal {
    NSLog(@" signal : %@", signal);
}
```

Swift:

```swift
func getWifiSignalStrength() {
    self.device?.getWifiSignalStrength(success: {
        print("get wifi signal strength success")
    }, failure: { (error) in
        if let e = error {
            print("get wifi signal strength failure: \(e)")
        }
    })
}

// MARK: - WiserSmartDeviceDelegate
func device(_ device: WiserSmartDevice!, signal: String!) {

}
```



### 获取网关下的子设备列表

Objc:

```objc
- (void)getSubDeviceList {
	// self.device = [WiserSmartDevice deviceWithDeviceId:@"your_device_id"];

	[self.device getSubDeviceListFromCloudWithSuccess:^(NSArray<WiserSmartDeviceModel *> *subDeviceList) {
        NSLog(@"get sub device list success");
    } failure:^(NSError *error) {
        NSLog(@"get sub device list failure: %@", error);
    }];
}
```

Swift:

```swift
func getSubDeviceList() {
    device?.getSubDeviceListFromCloud(success: { (subDeviceList) in
        print("get sub device list success")
    }, failure: { (error) in
        if let e = error {
            print("get sub device list failure: \(e)")
        }
    })
}
```



### 固件升级

**固件升级流程:**

获取设备升级信息 -> 下发联网模块升级指令 -> 联网模块升级成功 -> 下发设备控制模块升级指令 -> 设备控制模块升级成功

使用获取设备升级信息接口获取到的WiserSmartFirmwareUpgradeModel固件升级模型中，type属性能获取到固件的类型，typeDesc属性能获取到固件类型的描述。

#### 获取设备升级信息：

Objc:

```objc
- (void)getFirmwareUpgradeInfo {
	// self.device = [WiserSmartDevice deviceWithDeviceId:@"your_device_id"];

	[self.device getFirmwareUpgradeInfo:^(NSArray<WiserSmartFirmwareUpgradeModel *> *upgradeModelList) {
		NSLog(@"getFirmwareUpgradeInfo success");
	} failure:^(NSError *error) {
		NSLog(@"getFirmwareUpgradeInfo failure: %@", error);
	}];
}
```

Swift:

```swift
func getFirmwareUpgradeInfo() {
    device?.getFirmwareUpgradeInfo({ (upgradeModelList) in
        print("getFirmwareUpgradeInfo success")
    }, failure: { (error) in
        if let e = error {
            print("getFirmwareUpgradeInfo failure: \(e)")
        }
    })
}
```



#### 下发升级指令：

Objc:

```objc
- (void)upgradeFirmware {
	// self.device = [WiserSmartDevice deviceWithDeviceId:@"your_device_id"];
	// type: 从获取设备升级信息接口 getFirmwareUpgradeInfo 返回的固件类型
	// WiserSmartFirmwareUpgradeModel - type

	[self.device upgradeFirmware:type success:^{
		NSLog(@"upgradeFirmware success");
	} failure:^(NSError *error) {
		NSLog(@"upgradeFirmware failure: %@", error);
	}];
}
```

Swift:

```swift
func upgradeFirmware() {
    // type: 从获取设备升级信息接口 getFirmwareUpgradeInfo 返回的固件类型
    // WiserSmartFirmwareUpgradeModel - type
    device?.upgradeFirmware(type, success: {
        print("upgradeFirmware success")
    }, failure: { (error) in
        if let e = error {
            print("upgradeFirmware failure: \(e)")
        }
    })
}
```



#### 回调接口：

Objc:

```objc
- (void)deviceFirmwareUpgradeSuccess:(WiserSmartDevice *)device type:(NSInteger)type {
	//固件升级成功
}

- (void)deviceFirmwareUpgradeFailure:(WiserSmartDevice *)device type:(NSInteger)type {
	//固件升级失败
}

- (void)device:(WiserSmartDevice *)device firmwareUpgradeProgress:(NSInteger)type progress:(double)progress {
	//固件升级的进度
}

```

Swift:

```swift
func deviceFirmwareUpgradeSuccess(_ device: WiserSmartDevice!, type: Int) {
    //固件升级成功
}

func deviceFirmwareUpgradeFailure(_ device: WiserSmartDevice!, type: Int) {
    //固件升级失败
}

func device(_ device: WiserSmartDevice!, firmwareUpgradeProgress type: Int, progress: Double) {
    //固件升级的进度
}
```


