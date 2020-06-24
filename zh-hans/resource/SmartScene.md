## 智能场景

智能分为场景和自动化。场景是用户添加动作，手动触发；自动化是由用户设定条件，当条件触发后自动执行设定的动作。

**以下文档中手动场景和自动化场景简称为场景。**

施耐德云支持用户根据实际生活场景，通过设置气象或设备条件，当条件满足时，让一个或多个设备执行相应的任务。

场景相关的功能对应`WiserSmartScene`和`WiserSmartSceneManager`两个类，`WiserSmartScene`提供了单个场景的添加、编辑、删除、执行4种操作，需要使用场景id进行初始化，场景id指的是`WiserSmartSceneModel`的`sceneId`字段，可以从场景列表中获取。`WiserSmartSceneManager`类（单例）则主要提供了场景里条件、任务、设备、城市相关的所有数据。

在使用智能场景相关的接口之前，需要首先了解场景条件和场景任务这两个概念。

#### 场景条件

场景条件对应`WiserSmartSceneConditionModel`类，施耐德云支持以下条件类型：

- 气象条件：包括温度、湿度、天气、PM2.5、空气质量、日落日出，用户选择气象条件时，可以选择当前城市。
- 设备条件：指用户可预先选择一个设备的功能状态，当该设备达到该状态时，会触发当前场景里的任务，但同一设备不能同时作为条件和任务，避免操作冲突。
- 定时条件：指可以按照指定的时间去执行预定的任务。

#### 场景任务

场景任务是指当该场景满足已经设定的气象或设备条件时，让一个或多个设备执行某种操作，对应`WiserSmartSceneActionModel`类。或者关闭、开启一个自动化（带有场景条件的就叫做自动化）

**新增场景时，场景条件和场景任务对象的创建，参考本文档末尾的示例。**

#### 获取场景列表

Objc:

```objc
// 获取家庭下的场景列表
- (void)getSmartSceneList {
    [[WiserSmartSceneManager sharedInstance] getSceneListWithHomeId:homeId success:^(NSArray<WiserSmartSceneModel *> *list) {
        NSLog(@"get scene list success %@:", list);
    } failure:^(NSError *error) {
        NSLog(@"get scene list failure: %@", error);
    }];
}
```

Swift:

```swift
// 获取家庭下的场景列表
func getSmartSceneList() {
    WiserSmartSceneManager.sharedInstance()?.getSceneList(withHomeId: homeId, success: { (list) in
        print("get scene list success: \(list)")
    }, failure: { (error) in
        if let e = error {
            print("get scene list failure: \(e)")
        }
    })
}
```



#### 获取条件列表

获取条件列表，如温度、湿度、天气、PM2.5、日落日出等，注意：设备也可作为条件。
条件中的温度分为摄氏度和华氏度，根据需求传入需要的数据。

Objc:

```objc
- (void)getConditionList {
    [[WiserSmartSceneManager sharedInstance] getConditionListWithFahrenheit:YES success:^(NSArray<WiserSmartSceneDPModel *> *list) {
        NSLog(@"get condition list success:%@", list);
    } failure:^(NSError *error) {
        NSLog(@"get condition list failure: %@", error);
    }];
}
```

Swift:

```swift
func getConditionList() {
    WiserSmartSceneManager.sharedInstance()?.getConditionList(withFahrenheit: true, success: { (list) in
        print("get condition list success: \(list)")
    }, failure: { (error) in
        if let e = error {
            print("get condition list failure: \(e)")
        }
    })
}
```



#### 获取任务设备列表

添加任务时，需获取任务的设备列表，用来选择执行相应的任务。

Objc:

```objc
- (void)getActionDeviceList {
	[[WiserSmartSceneManager sharedInstance] getActionDeviceListWithHomeId:homeId success:^(NSArray<WiserSmartDeviceModel *> *list) {
		NSLog(@"get action device list success:%@", list);
	} failure:^(NSError *error) {
		NSLog(@"get action device list failure: %@", error);
	}];
}
```

Swift:

```swift
func getActionDeviceList() {
    WiserSmartSceneManager.sharedInstance()?.getActionDeviceList(withHomeId: homeId, success: { (list) in
        print("get action device list success: \(list)")
    }, failure: { (error) in
        if let e = error {
            print("get action device list failure: \(e)")
        }
    })
}
```



#### 获取条件设备列表

添加条件时，除了温度、湿度、天气等这些气象条件可以作为任务执行条件外，设备也可以作为条件，即获取条件设备列表，然后选择一个设备执行相应的任务作为条件。

Objc:

```objc
- (void)getConditionDeviceList {
	[[WiserSmartSceneManager sharedInstance] getConditionDeviceListWithHomeId:homeId success:^(NSArray<WiserSmartDeviceModel *> *list) {
		NSLog(@"get condition device list success:%@", list);
	} failure:^(NSError *error) {
		NSLog(@"get condition device list failure: %@", error);
	}];
}
```

Swift:

```swift
func getConditionDeviceList() {
    WiserSmartSceneManager.sharedInstance()?.getConditionDeviceList(withHomeId: homeId, success: { (list) in
        print("get condition device list success: \(list)")
    }, failure: { (error) in
        if let e = error {
            print("get condition device list failure: \(e)")
        }
    })
}
```



#### 获取任务设备的dp列表

添加或编辑场景任务时，选择设备后，需要根据选择设备的deviceId获取设备dp列表，进而选择某一个dp功能点，即指定该设备执行该项任务。

Objc:

```objc
- (void)getActionDeviceDPList {
	[[WiserSmartSceneManager sharedInstance] getActionDeviceDPList:@"your_device_id" success:^(NSArray<WiserSmartSceneDPModel *> *list) {
		NSLog(@"get action device dp list success:%@", list);
	} failure:^(NSError *error) {
		NSLog(@"get action device dp list failure: %@", error);
	}];
}
```

Swift:

```swift
func getActionDeviceDPList() {
    WiserSmartSceneManager.sharedInstance()?.getActionDeviceDPList(withDevId: "your_device_id", success: { (list) in
        print("get action device dp list success: \(list)")
    }, failure: { (error) in
        if let e = error {
            print("get action device dp list failure: \(e)")
        }
    })
}
```



#### 获取条件设备的dp列表

选择场景条件时，选择了设备，需要根据选择设备的deviceId获取设备dp列表，进而选择某一个dp功能点，即指定该设备执行该dp功能作为该场景的执行条件。

Objc:

```objc
- (void)getCondicationDeviceDPList {
	[[WiserSmartSceneManager sharedInstance] getCondicationDeviceDPList:@"your_device_id" success:^(NSArray<WiserSmartSceneDPModel *> *list) {
		NSLog(@"get condition device dp list success:%@", list);
	} failure:^(NSError *error) {
		NSLog(@"get condition device dp list failure: %@", error);
	}];
}
```

Swift:

```swift
func getCondicationDeviceDPList() {
    WiserSmartSceneManager.sharedInstance()?.getCondicationDeviceDPList(withDevId: "your_device_id", success: { (list) in
        print("get condition device dp list success: \(list)")
    }, failure: { (error) in
        if let e = error {
            print("get condition device dp list failure: \(e)")
        }
    })
}
```



#### 获取城市列表

选择场景气象条件时，根据国家码获取城市列表，用户可以选择当前城市。（注：国外部分国家的城市列表可能暂时不全，如果是国外用户，建议根据经纬度获取城市信息。）countryCode使用isoCountryCode，例如中国="CN"。

Objc:

```objc
- (void)getCityList {
	[[WiserSmartSceneManager sharedInstance] getCityListWithCountryCode:@"your_country_code" success:^(NSArray<WiserSmartCityModel *> *list) {
		NSLog(@"get city list success:%@", list);
	} failure:^(NSError *error) {
   		NSLog(@"get city list failure: %@", error);
	}];
}
```

Swift:

```swift
func getCityList() {
    WiserSmartSceneManager.sharedInstance()?.getCityList(withCountryCode: "your_country_code", success: { (list) in
        print("get city list success: \(list)")
    }, failure: { (error) in
        if let e = error {
            print("get city list failure: \(e)")
        }
    })
}
```



#### 根据经纬度获取城市信息

Objc:

```objc
- (void)getCityInfo {
	[[WiserSmartSceneManager sharedInstance] getCityInfo:@"your_location_latitude" longitude:@"your_location_longitude" success:^(WiserSmartCityModel *city) {
		NSLog(@"get city info success:%@", city);
	} failure:^(NSError *error) {
		NSLog(@"get city info failure:%@", error);
	}];
}
```

Swift:

```swift
func getCityInfo() {
    WiserSmartSceneManager.sharedInstance()?.getCityInfo(withLatitude: "your_location_latitude", longitude: "your_location_longitude", success: { (city) in
        print("get city info success: \(city)")
    }, failure: { (error) in
        if let e = error {
            print("get city info failure: \(e)")
        }
    })
}
```



#### 根据城市id获取城市信息

根据城市id获取城市信息，城市id可以从城市列表获取。
Objc:

```objc
- (void) getCityInfo {
	[[WiserSmartSceneManager sharedInstance] getCityInfoByCityId:@"your_city_id" success:^(WiserSmartCityModel *city) {
		NSLog(@"get city info success:%@", city);
	} failure:^(NSError *error) {
		NSLog(@"get city info failure:%@", error);
	}];
}
```

Swift:

```swift
func getCityInfo() {
    WiserSmartSceneManager.sharedInstance()?.getCityInfo(withCityId: "your_city_id", success: { (city) in
        print("get city info success: \(city)")
    }, failure: { (error) in
        if let e = error {
            print("get city info failure: \(e)")
        }
    })
}
```



#### 场景排序

Objc:

```objc
- (void) sortScene {
	[[WiserSmartSceneManager sharedInstance] sortSceneWithHomeId:homeId sceneIdList:(NSArray<NSString *> *) success:^{
        NSLog(@"sort scene success");
    } failure:^(NSError *error) {
        NSLog(@"sort scene failure:%@", error);
    }];
}
```

Swift:

```swift
func sortScene() {
    WiserSmartSceneManager.sharedInstance()?.sortScene(withHomeId: homeId, sceneIdList: ["sceneId list"], success: {
        print("sort scene success")
    }, failure: { (error) in
        if let e = error {
            print("sort scene failure: \(e)")
        }
    })
}
```

#### 获取场景背景图片列表
获取场景支持的背景图片url列表。

Objc:

```objc
- (void)getDefaultSceneCover {
	[[WiserSmartSceneManager sharedInstance] getSmartSceneBackgroundCoverWithsuccess:^(NSArray *list) {
        
    } failure:^(NSError *error) {
        
    }];
}
```

Swift:

```swift
func getDefaultSceneCover() {
    WiserSmartSceneManager.sharedInstance()?.getSmartSceneBackgroundCover(withsuccess: {(list) in
        
    }, failure: { (error) in
       
    })
}
```



## 单个场景操作

`WiserSmartScene`提供了单个场景的添加、编辑、删除、执行4种操作，需要使用场景id进行初始化，场景id指的是`WiserSmartSceneModel`的`sceneId`字段，可以从场景列表中获取。


#### 添加场景

添加场景需要传入场景名称，家庭的Id，背景图片的url，是否显示在首页，条件列表，任务列表（至少一个任务），满足任一条件还是满足所有条件时执行。也可以只设置名称和任务，背景图片，不设置条件，但是需要手动执行。

Objc:


```objc
- (void)addSmartScene {

    [WiserSmartScene addNewSceneWithName:@"your_scene_name" homeId:homeId background:@"background_url" showFirstPage:YES conditionList:(NSArray<WiserSmartSceneConditionModel *> *) actionList:(NSArray<WiserSmartSceneActionModel *> *) matchType:WiserSmartConditionMatchAny success:^(WiserSmartSceneModel *sceneModel) {
        NSLog(@"add scene success %@:", sceneModel);
    } failure:^(NSError *error) {
        NSLog(@"add scene failure: %@", error);
    }];
}

```
Swift:

```swift
func addSmartScene() {
    WiserSmartScene.addNewScene(withName: "your_scene_name", homeId: homeId, background: "background_url", showFirstPage: true, conditionList: [WiserSmartSceneConditionModel]!, actionList: [WiserSmartSceneActionModel]!, matchType: WiserSmartConditionMatchAny, success: { (sceneModel) in
        print("add scene success :\(sceneModel)")
    }) { (error) in
        if let e = error {
            print("add scene failure: \(e)")
        }
    }
}
```



#### 编辑场景

编辑场景的名称、背景图、条件列表、任务列表、满足任一条件还是满足所有条件时执行

Objc:

```objc
- (void)modifySmartScene {
//    self.smartScene = [WiserSmartScene sceneWithSceneId:@"your_scene_id"];
    [self.smartScene modifySceneWithName:name background:@"background_url" showFirstPage:YES condition:(NSArray<WiserSmartSceneConditionModel *> *) actionList:(NSArray<WiserSmartSceneActionModel *> *) matchType:WiserSmartConditionMatchAny success:^{
        NSLog(@"modify scene success");
    } failure:^(NSError *error) {
        NSLog(@"modify scene failure: %@", error);
    }];
}
```
Swift:

```swift
func modifySmartScene() {
    smartScene?.modifyScene(withName: "name", background: "background_url", showFirstPage: true, conditionList: [WiserSmartSceneConditionModel]!, actionList: [WiserSmartSceneActionModel]!, matchType: WiserSmartConditionMatchAny, success: {
        print("modify scene success")
    }, failure: { (error) in
        if let e = error {
            print("modify scene failure: \(e)")
        }
    })
}
```



#### 删除场景

Objc:

```objc
- (void)deleteSmartScene {
//    self.smartScene = [WiserSmartScene sceneWithSceneId:@"your_scene_id"];
    [self.smartScene deleteSceneWithSuccess:^{
        NSLog(@"delete scene success");
    } failure:^(NSError *error) {
        NSLog(@"delete scene failure: %@", error);
    }];
}
```
Swift:

```swift
func deleteSmartScene() {
    smartScene?.delete(success: {
        print("delete scene success")
    }, failure: { (error) in
        if let e = error {
            print("delete scene failure: \(e)")
        }
    })
}
```



#### 执行场景

Objc:

```objc
- (void)executeSmartScene {
//    self.smartScene = [WiserSmartScene sceneWithSceneId:@"your_scene_id"];
	[self.smartScene executeSceneWithSuccess:^{
   		NSLog(@"execute scene success");
    } failure:^(NSError *error) {
        NSLog(@"execute scene failure: %@", error);
    }];
}
```

Swift:

```swift
func executeSmartScene() {
    smartScene?.execute(success: {
        print("execute scene success")
    }, failure: { (error) in
        if let e = error {
            print("execute scene failure: \(e)")
        }
    })
}
```



#### 开启场景（只有至少带有至少一个条件的场景才可以开启和失效场景）

Objc:

```objc
- (void)enableSmartScene {
//    self.smartScene = [WiserSmartScene sceneWithSceneId:@"your_scene_id"];
	[self.smartScene enableSceneWithSuccess:^{
   		NSLog(@"enable scene success");
    } failure:^(NSError *error) {
        NSLog(@"enable scene failure: %@", error);
    }];
}
```

Swift:

```swift
func enableSmartScene() {
    smartScene?.enable(success: {
        print("enable scene success")
    }, failure: { (error) in
        if let e = error {
            print("enable scene failure: \(e)")
        }
    })
}
```



#### 失效场景（只有至少带有至少一个条件的场景才可以开启和失效场景）

Objc:

```objc
- (void)disableSmartScene {
//    self.smartScene = [WiserSmartScene sceneWithSceneId:@"your_scene_id"];
	[self.smartScene disableSceneWithSuccess:^{
   		NSLog(@"disable scene success");
    } failure:^(NSError *error) {
        NSLog(@"disable scene failure: %@", error);
    }];
}
```
Swift:

```swift
func disableSmartScene() {
    smartScene?.disableScene(success: {
        print("disable scene success")
    }, failure: { (error) in
        if let e = error {
            print("disable scene failure: \(e)")
        }
    })
}
```



## 场景条件和场景动作对象创建示例

SDK在3.14.0及以上版本加入了WiserSmartSceneDataFactory这个工具类集合，用于便捷的创建场景的条件、动作、生效时间段条件。

如果使用的是3.14.0以前的版本，请参照以下示例创建条件和动作。
如果使用的是3.14.0及以上版本，推荐使用WiserSmartSceneDataFactory提供的工具类创建条件和动作。

#### 场景条件

##### 创建场景条件对象WiserSmartSceneConditionModel
气象条件包括温度、湿度、天气、PM2.5、空气质量、日落日出，这里以设置温度条件为例创建气象条件对象。场景条件也可以设置定时条件和设备条件。

从“获取条件列表”接口可以获取到所有气象条件的`WiserSmartSceneDPModel`对象列表。可以根据`WiserSmartSceneDPModel`对象的`entityName`和`entityId`区分不同的气象条件。从获取城市信息相关的接口，获取到`WiserSmartCityModel`对象，使用其中的`cityId`值作为定位信息。

选择完具体的条件值之后,如果将界面选择的温度、城市等信息保存在了`WiserSmartSceneDPModel`对象中(也可以存在任何你喜欢的对象中)，可以通过一个`WiserSmartSceneDPModel`对象初始化一个`WiserSmartSceneConditionModel`条件对象，示例方法如下，这里使用了Category为`WiserSmartSceneConditionModel`增加了一个分类方法：


	//新增初始化方法
	- (instancetype)initWithSmartSceneDPModel:(WiserSmartSceneDPModel *)dpModel {
	    
	    if (self = [super init]) {
	        self.entityType = dpModel.entityType;
	        self.iconUrl = dpModel.iconUrl;
	        if (dpModel.entityType == 3) {
	        	//气象条件
	            self.entityId = dpModel.cityId;
	            self.entityName = dpModel.cityName;
	            self.entitySubIds = dpModel.entitySubId;
	            self.cityName = dpModel.cityName;
	            self.cityLatitude = dpModel.cityLatitude;
	            self.cityLongitude = dpModel.cityLongitude;
	        } else if (dpModel.entityType == 7) {
	        	//定时条件
	            NSString *value = dpModel.valueRangeJson[dpModel.selectedRow][0];
	            self.extraInfo = @{@"delayTime" : value};
	        } else {
	        	//设备条件
	            self.entityId = dpModel.devId;
	            WiserSmartDevice *device = [WiserSmartDevice deviceWithDeviceId:dpModel.devId];
	            self.entityName = device.deviceModel.name;
	            self.entitySubIds = [NSString stringWithFormat:@"%ld", (long)dpModel.dpId];
	        }
	        //expr数组的组装见下文
	        self.expr = dpModel.expr;
	    }
	    return self;
	}


##### expr 表达式组装

`WiserSmartSceneConditionModel`的expr属性描述了条件的表达式，如“温度低于15℃”这样的一个条件，就可以用expr来描述。expr是一个数组（这里要注意，最外层一定是数组），数组中的每一个对象描述了一个条件，如@[@"$temp",@"<",@15]这个条件数组就描述了温度低于15℃这个条件。注意，每个气象条件都应该对应一个`WiserSmartSceneConditionModel`，所以expr数组中只包含一个条件数组。

气象条件expr示例：

- 温度 @[@[@"$temp",@"<",@15]]
- 湿度 @[@[@"$humidity",@"==",@"comfort"]]
- 天气 @[@[@"$condition",@"==",@"snowy"]]
- PM2.5 @[@[@"$pm25",@"==",@"fine"]]
- 空气质量 @[@[@"$aqi",@"==",@"fine"]]
- 日出日落 @[@[@"$sunsetrise",@"==",@"sunrise"]]

定时条件expr示例:

定时条件使用一个字典表示，例如{timeZoneId = "Asia/Shanghai",loops = "0000000",time = "08:00",date = "20180308"}。其中loops中的每一位分别表示周日到周六的每一天，1表示生效，0表示不生效。注意这个表示定时的字典也需要使用数组包起来，因为expr是个数组。

设备条件expr示例：

设备条件使用一个数组表示选定的条件值。选择的条件组装的expr可以表示为@[@[@"$dp1",@"==",@YES]],这里可以表示一个“电灯开”的条件。其中`dp1`是`WiserSmartSceneDPModel`中提供的dp点的名称。
####场景动作
场景动作类是`WiserSmartSceneActionModel`，其中的‘actionExecutor’属性即表示场景动作类型。场景动作类型包括:

- dpIssue 设备
- deviceGroupDpIssue 群组
- ruleTrigger 触发场景
- ruleEnable 启动自动化
- ruleDisable 禁用自动化
- delay 延时动作

新建完`WiserSmartSceneActionModel`的对象后，分别设置对应的属性，重点关注三个属性entityId、actionExecutor、executorProperty，这三个属性描述了哪个对象要做动作，做什么类型的动作，具体做什么动作。

1. 设备。entityId属性对应设备的devId，actionExecutor是dpIssue，executorProperty是一个字典，比如{"1":YES},表示dp点“1”，执行的动作是YES，这个可以表示灯打开等布尔类型的dp点。dp点的Id和dp可取的值可以通过“获取任务设备的dp列表”接口获取到。
2. 群组。entityId属性对应群组的groupId, actionExecutor是deviceGroupDpIssue，其他的属性和设备动作相同。
3. 触发场景。entityId是场景sceneId，actionExecutor是ruleTrigger，executorProperty不需要。
4. 启动自动化。entityId是自动化sceneId，actionExecutor是ruleEnable，executorProperty不需要。
5. 禁用自动化。entityId是自动化sceneId，actionExecutor是ruleDisable，executorProperty不需要。
6. 延时动作。entityId是delay，actionExecutor是delay，executorProperty是延时的时间，用一个字典表示，形式和key为{@"minutes":@"1",@"seconds":@"30"},表示1分30秒。目前最大支持59分59秒。

#### WiserSmartSceneDataFactory工具类集合

WiserSmartSceneDataFactory中包涵以下创建工具类：

- WiserSmartScenePreConditionFactory.h 用于创建自动化场景的前置条件，如生效时间段。
- WiserSmartSceneConditionFactory.h 用于创建自动化场景的条件，如天气条件、设备条件。
- WiserSmartSceneActionFactory.h 用于创建场景动作，如设备动作。

以及两个辅助类：

- WiserSmartSceneExprModel.h 用于储存场景条件中的expr表达式。
- WiserSmartSceneConditionExprBuilder.h 自动化场景中条件表达式的生成工具类。

生效时间段、条件、动作的创建，所有支持的类型可以参照SDK头文件中的注释使用。注意：因为要适配多语言，条件和动作中，未生成用来显示条件和动作的详情的`exprDisplay`和`actionDisplayNew`，需要开发者根据条件中的表达式`expr`和动作中的执行参数`executorProperty`手动拼接生成。

以创建一个设备条件为例，使用顺序如下：
1. 使用`WiserSmartSceneConditionExprBuilder`创建一个`WiserSmartSceneExprModel`对象，生成创建条件所需的表达式`expr`。
2. 使用`WiserSmartSceneConditionFactory`中的API，传入第一步中生成的`WiserSmartSceneExprModel`对象以及其他必需参数，生成条件对象。

生成前置条件和动作直接使用`WiserSmartScenePreConditionFactory`和`WiserSmartSceneActionFactory`中提供的API即可。
