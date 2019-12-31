## 家庭管理

用户登录成功后需要通过`WiserSmartHomeManager`去获取整个家庭列表的信息,然后初始化其中的一个家庭`WiserSmartHome`，获取家庭详情信息，对家庭中的设备进行管理，控制。


### 家庭列表信息变化的回调

实现`WiserSmartHomeManagerDelegate`代理协议后，可以在家庭列表更变的回调中进行处理。

Objc:

```objc
#pragma mark - WiserSmartHomeManagerDelegate


// 添加一个家庭
- (void)homeManager:(WiserSmartHomeManager *)manager didAddHome:(WiserSmartHomeModel *)home {

}

// 删除一个家庭
- (void)homeManager:(WiserSmartHomeManager *)manager didRemoveHome:(long long)homeId {

}

// MQTT连接成功
- (void)serviceConnectedSuccess {
    // 刷新当前家庭UI
}
```

Swift:

```swift
extension ViewController: WiserSmartHomeManagerDelegate {
    
    // 添加一个家庭
    func homeManager(_ manager: WiserSmartHomeManager!, didAddHome home: WiserSmartHomeModel!) {
        
    }
    
    // 删除一个家庭
    func homeManager(_ manager: WiserSmartHomeManager!, didRemoveHome homeId: Int64) {
        
    }
    
    // MQTT连接成功
    func serviceConnectedSuccess() {
        // 刷新当前家庭UI
    }
    
}
```



### 获取家庭列表

获取家庭列表，返回数据只是家庭的简单信息。如果要获取具体家庭的详情，需要去初始化一个home，调用接口 `getHomeDetailWithSuccess:failure:`

Objc:

```objc
- (void)getHomeList {

	[self.homeManager getHomeListWithSuccess:^(NSArray<WiserSmartHomeModel *> *homes) {
        // homes 家庭列表
    } failure:^(NSError *error) {
        NSLog(@"get home list failure: %@", error);
    }];
}
```

Swift:

```swift
let homeManager: WiserSmartHomeManager = WiserSmartHomeManager()

func getHomeList() {
    homeManager.getHomeList(success: { (homes) in
        // homes 家庭列表
    }) { (error) in
        if let e = error {
            print("get home list failure: \(e)")
        }
    }
}
```



### 添加家庭

Objc:

```objc
- (void)addHome {

    [self.homeManager addHomeWithName:@"you_home_name"
                          geoName:@"city_name"
                            rooms:@[@"room_name"]
                         latitude:lat
                        longitude:lon
                          success:^(double homeId) {

        // homeId 创建的家庭的homeId
        NSLog(@"add home success");
    } failure:^(NSError *error) {
        NSLog(@"add home failure: %@", error);
    }];
}
```

Swift:

```swift
 func addHome() {
    homeManager.addHome(withName: "you_home_name", geoName: "city_name", rooms: ["room_name"], latitude: lat, longitude: lon, success: { (homeId) in
        // homeId 创建的家庭的homeId
        print("add home success")
    }) { (error) in
        if let e = error {
            print("add home failure: \(e)")
        }
    }
}
```





## 家庭操作

主要功能：用来获取和修改，解散家庭。获取，添加和删除家庭的成员。新增，解散房间，房间进行排序。

单个家庭信息管理相关的所有功能对应`WiserSmartHome`类，需要使用家庭Id进行初始化。错误的家庭Id可能会导致初始化失败，返回`nil`。

初始化 home 对象之后需要获取家庭的详情接口（getHomeDetailWithSuccess:failure:），home 实例对象中的属性 homeModel,roomList,deviceList,groupList 才有数据。


### 单个家庭信息变化的回调

实现`WiserSmartHomeDelegate`代理协议后，可以在单个家庭信息更变的回调中进行处理。

Objc:

```objc
- (void)initHome {
    self.home = [WiserSmartHome homeWithHomeId:homeId];
    self.home.delegate = self;
}

#pragma mark - WiserSmartHomeDelegate

// 家庭的信息更新，例如name
- (void)homeDidUpdateInfo:(WiserSmartHome *)home {
    [self reload];
}

// 我收到的共享设备列表变化
- (void)homeDidUpdateSharedInfo:(WiserSmartHome *)home {
    [self reload];
}

// 家庭下新增房间代理回调
- (void)home:(WiserSmartHome *)home didAddRoom:(WiserSmartRoomModel *)room {
  	[self reload];
}

// 家庭下删除房间代理回调
- (void)home:(WiserSmartHome *)home didRemoveRoom:(long long)roomId {
  	[self reload];
}

// 房间信息变更，例如name
- (void)home:(WiserSmartHome *)home roomInfoUpdate:(WiserSmartRoomModel *)room {
    [self reload];
}

// 房间与设备，群组的关系变化
- (void)home:(WiserSmartHome *)home roomRelationUpdate:(WiserSmartRoomModel *)room {
    [self reload];
}

// 添加设备
- (void)home:(WiserSmartHome *)home didAddDeivice:(WiserSmartDeviceModel *)device {
    [self reload];
}

// 删除设备
- (void)home:(WiserSmartHome *)home didRemoveDeivice:(NSString *)devId {
    [self reload];
}

// 设备信息更新，例如name
- (void)home:(WiserSmartHome *)home deviceInfoUpdate:(WiserSmartDeviceModel *)device {
    [self reload];
}

// 家庭下设备的 dps 变化代理回调
- (void)home:(WiserSmartHome *)home device:(WiserSmartDeviceModel *)device dpsUpdate:(NSDictionary *)dps {
    [self reload];
}

// 添加群组
- (void)home:(WiserSmartHome *)home didAddGroup:(WiserSmartGroupModel *)group {
    [self reload];
}

// 删除群组
- (void)home:(WiserSmartHome *)home didRemoveGroup:(NSString *)groupId {
    [self reload];
}

// 群组信息更新，例如name
- (void)home:(WiserSmartHome *)home groupInfoUpdate:(WiserSmartGroupModel *)group {
    [self reload];
}

// 家庭下群组 dps 变化代理回调
- (void)home:(WiserSmartHome *)home group:(WiserSmartGroupModel *)group dpsUpdate:(NSDictionary *)dps {
    [self reload];
}

// the delegate of warning information update
// 家庭下设备的告警信息变化的代理回调
- (void)home:(WiserSmartHome *)home device:(WiserSmartDeviceModel *)device warningInfoUpdate:(NSDictionary *)warningInfo {
	//...  
}

// the delegate of device firmware upgrade status update
// 家庭下设备升级状态的回调
- (void)home:(WiserSmartHome *)home device:(WiserSmartDeviceModel *)device upgradeStatus:(WiserSmartDeviceUpgradeStatus)upgradeStatus {
  	//...
}
```

Swift:

```swift
var home: WiserSmartHome?

extension ViewController: WiserSmartHomeDelegate {
  
  func initHome() {
      home = WiserSmartHome(homeId: homeId)
      home?.delegate = self
  }
  
  // 家庭的信息更新，例如name
  func homeDidUpdateInfo(_ home: WiserSmartHome!) {
//        reload()
  }

  // 我收到的共享设备列表变化
  func homeDidUpdateSharedInfo(_ home: WiserSmartHome!) {

  }

  // 家庭下新增房间代理回调
  func home(_ home: WiserSmartHome!, didAddRoom room: WiserSmartRoomModel!) {
      //...
  }

  // 家庭下删除房间代理回调
  func home(_ home: WiserSmartHome!, didRemoveRoom roomId: int32!) {
      //...
  }

  // 房间信息变更，例如name
  func home(_ home: WiserSmartHome!, roomInfoUpdate room: WiserSmartRoomModel!) {
//        reload()/
  }

  // 房间与设备，群组的关系变化
  func home(_ home: WiserSmartHome!, roomRelationUpdate room: WiserSmartRoomModel!) {

  }

  // 添加设备
  func home(_ home: WiserSmartHome!, didAddDeivice device: WiserSmartDeviceModel!) {

  }

  // 删除设备
  func home(_ home: WiserSmartHome!, didRemoveDeivice devId: String!) {

  }

  // 设备信息更新，例如name
  func home(_ home: WiserSmartHome!, deviceInfoUpdate device: WiserSmartDeviceModel!) {

  }

  // 家庭下设备的 dps 变化代理回调
  func home(_ home: WiserSmartHome!, device: WiserSmartDeviceModel!, dpsUpdate dps: [AnyHashable : Any]!) {
      //...
  }

  // 添加群组
  func home(_ home: WiserSmartHome!, didAddGroup group: WiserSmartGroupModel!) {

  }

  // 删除群组
  func home(_ home: WiserSmartHome!, didRemoveGroup groupId: String!) {

  }

  // 群组信息更新，例如name
  func home(_ home: WiserSmartHome!, groupInfoUpdate group: WiserSmartGroupModel!) {

  }

  // 家庭下群组的 dps 变化代理回调
  func home(_ home: WiserSmartHome!, group: WiserSmartGroupModel!, dpsUpdate dps: [AnyHashable : Any]!) {
			//...
  }
  
  // 家庭下设备的告警信息变化的代理回调
  func home(_ home: WiserSmartHome!, device: WiserSmartDeviceModel!, warningInfoUpdate warningInfo: [AnyHashable : Any]!) {
    	//...
  }
  
  // 家庭下设备升级状态的回调
  func home(_ home: WiserSmartHome!, device: WiserSmartDeviceModel!, upgradeStatus status WiserSmartDeviceUpgradeStatus) {
    	//....
  }
  
}
```



### 获取家庭的信息

初始化 home 对象之后需要获取家庭的详情， home 对象的属性 homeModel,roomList,deviceList,groupList 才有数据

Objc:

```objc
- (void)getHomeDetailInfo {

	[self.home getHomeDetailWithSuccess:^(WiserSmartHomeModel *homeModel) {
        // homeModel 家庭信息
        NSLog(@"get home detail success");
    } failure:^(NSError *error) {
        NSLog(@"get home detail failure: %@", error);
    }];
}
```

Swift:

```swift
func getHomeDetailInfo() {
    home?.getDetailWithSuccess({ (homeModel) in
        print("get home detail success")
    }, failure: { (error) in
        if let e = error {
            print("get home detail failure: \(e)")
        }
    })
}
```



### 修改家庭信息

Objc:

```objc
- (void)updateHomeInfo {
    [self.home updateHomeInfoWithName:@"new_home_name" geoName:@"city_name" latitude:lat longitude:lon success:^{
        NSLog(@"update home info success");
    } failure:^(NSError *error) {
        NSLog(@"update home info failure: %@", error);
    }];
}

```

Swift:

```swift
func updateHomeInfo() {
    home?.updateInfo(withName: "new_home_name", geoName: "city_name", latitude: lat, longitude: lon, success: {
        print("update home info success")
    }, failure: { (error) in
        if let e = error {
            print("update home info failure: \(e)")
        }
    })
}
```

###对家庭下设备和群组进行排序

Objc:

```objc
// orderList: [@{@"bizId": @"XXX", @"bizType": @"XXX"},@{@"bizId": @"XXX",@"bizType": @"XXX"}] 其中bizId为设备的devId或群组的groupId, device的bizType = @"6" group的bizType = @"5"
- (void)sortDeviceOrGroupWithOrderList:(NSArray<NSDictionary *> *)orderList {
	[self.home sortDeviceOrGroupWithOrderList:orderList success:^ {
        NSLog(@"sort device or group success");
    } failure:^(NSError *error) {
        NSLog(@"sort device or group failure: %@", error);
    }];
}
```

Swift:

```swift
func sortDeviceOrGroup(withOrderList orderList: [[AnyHashable : Any]]?) {
    home.sortDeviceOrGroup(withOrderList: orderList, success: {
        print("sort device or group success")
    }, failure: { error in
        if let error = error {
            print("sort device or group failure: \(error)")
        }
    })
}
```

### 解散家庭

Objc:

```objc
- (void)dismissHome {

	[self.home dismissHomeWithSuccess:^() {
        NSLog(@"dismiss home success");
    } failure:^(NSError *error) {
        NSLog(@"dismiss home failure: %@", error);
    }];
}
```

Swift:

```swift
func dismissHome() {
    home?.dismiss(success: {
        print("dismiss home success")
    }, failure: { (error) in
        if let e = error {
            print("dismiss home failure: \(e)")
        }
    })
}
```



### 新增房间

Objc:

```objc
- (void)addHomeRoom {
    [self.home addHomeRoomWithName:@"room_name" success:^{
        NSLog(@"add room success");
    } failure:^(NSError *error) {
        NSLog(@"add room failure: %@", error);
    }];
}
```

Swift:

```swift
func addHomeRoom() {
    home?.addRoom(withName: "room_name", success: {
        print("add room success")
    }, failure: { (error) in
        if let e = error {
            print("add room failure: \(e)")
        }
    })
}
```



### 删除房间

Objc:

```objc
- (void)removeHomeRoom {
    [self.home removeHomeRoomWithRoomId:roomId success:^{
        NSLog(@"remove room success");
    } failure:^(NSError *error) {
        NSLog(@"remove room failure: %@", error);
    }];
}
```

Swift:

```swift
func removeHomeRoom() {
    home?.removeRoom(withRoomId: roomId, success: {
        print("remove room success")
    }, failure: { (error) in
        if let e = error {
            print("remove room failure: \(e)")
        }
    })
}
```



### 房间排序

Objc:

```objc
- (void)sortHomeRoom {
    [self.home sortRoomList:(NSArray<WiserSmartRoomModel *> *) success:^{
        NSLog(@"sort room success");
    } failure:^(NSError *error) {
        NSLog(@"sort room failure: %@", error);
    }];
}
```

Swift:

```swift
func sortHomeRoom() {
    home?.sortRoomList([WiserSmartRoomModel]!, success: {
        print("sort room success")
    }, failure: { (error) in
        if let e = error {
            print("sort room failure: \(e)")
        }
    })
}
```

### 家庭成员管理

家庭成员管理相关的所有功能对应`WiserSmartHome`和 `WiserSmartHomeMember`类，成员角色类型TYHomeRoleType

#### 添加家庭成员

拥有者(TYHomeRoleType_Owner) 可以添加管理员及以下角色，管理员(TYHomeRoleType_Admin)仅仅可以添加普通成员及以下角色

Objc:

```objc
- (void)addShare {
  //	_home = [WiserSmartHome homeWithHomeId:homeId];
    [_home addHomeMemberWithName:@"name" headPic:image countryCode:@"your_country_code" userAccount:@"account" role:TYHomeRoleType_Admin success:^(NSDictionary *dict) {
        NSLog(@"addNewMember success");
    } failure:^(NSError *error) {
        NSLog(@"addNewMember failure: %@", error);
    }];
}
```

Swift:

```swift
func addShare() {
    home?.addHomeMember(withName:@"name", headPic:image, countryCode: "your_country_code", account: "account", role: TYHomeRoleType_Admin, success: {
        print("addNewMember success")
    }, failure: { (error) in
        if let e = error {
            print("addNewMember failure: \(e)")
        }
    })
}
```



#### 获取家庭成员列表

Objc:

```objc
- (void)initMemberList {
  //	_home = [WiserSmartHome homeWithHomeId:homeId];
    [_home getHomeMemberListWithSuccess:^(NSArray<WiserSmartHomeMemberModel *> *memberList) {
        NSLog(@"getMemberList success: %@", memberList);
    } failure:^(NSError *error) {
        NSLog(@"getMemberList failure");
    }];
}
```

Swift:

```swift
func initMemberList() {
    home.getHomeMemberList(withSuccess: { memberList in
        print("getMemberList success: \(memberList)")
    }, failure: { (error) in
        if let e = error {
            print("getMemberList failure: \(e)")
        }
    })
}
```



#### 修改家庭成员的备注名称和成员角色

拥有者(TYHomeRoleType_Owner) 可以修改管理员及以下角色，管理员(TYHomeRoleType_Admin)仅仅可以修改普通成员及以下角色

Objc:

```objc
- (void)modifyMemberName:(WiserSmartHomeMemberModel *)memberModel name:(NSString *)name {
	// self.homeMember = [[WiserSmartHomeMember alloc] init];
	
  WiserSmartHomeMemberRequestModel *requestModel = [[WiserSmartHomeMemberRequestModel alloc] init];
	[self.homeMember updateHomeMemberInfoWithMemberRequestModel:requestModel  success:^{
        NSLog(@"modifyMemberName success");
    } failure:^(NSError *error) {
        NSLog(@"modifyMemberName failure: %@", error);
    }];
}
```

Swift:

```swift
func modifyMember(_ memberModel: WiserSmartHomeMemberModel, name: String) {
    homeMember?.updateHomeMemberName(withMemberRequestModel:requestModel, success: {
        print("modifyMemberName success")
    }, failure: { (error) in
        if let e = error {
            print("modifyMemberName failure: \(e)")
        }
    })
}
```



#### 删除家庭成员

拥有者(TYHomeRoleType_Owner) 可以删除管理员及以下角色，管理员(TYHomeRoleType_Admin)仅仅可以删除普通成员及以下角色

若成员传入自身memberId，家庭管理员，普通成员，自定义角色，调用此接口为离开家庭，此时该家庭未解散，设备也不会被重置；拥有者为解散家庭，同时该家庭下所有设备会被重置，效果与上文解散家庭一致。

Objc:

```objc
- (void)removeMember:(WiserSmartHomeMemberModel *)memberModel {
	// self.homeMember = [[WiserSmartHomeMember alloc] init];

	[self.homeMember removeHomeMemberWithMemberId:memberModel.memberId success:^{
        NSLog(@"removeMember success");
    } failure:^(NSError *error) {
        NSLog(@"removeMember failure: %@", error);
    }];
}
```

Swift:

```swift
func removeMember(_ memberModel: WiserSmartHomeMemberModel) {
    homeMember?.removeHomeMember(withMemberId: memberModel.memberId, success: {
        print("removeMember success")
    }, failure: { (error) in
        if let e = error {
            print("removeMember failure: \(e)")
        }
    })
}
```



####  接受或者拒绝家庭邀请

成员是否接受该家庭的邀请对应WiserSmartHomeModel下的dealStatus，受邀状态会分别对应TYHomeStatusPending、TYHomeStatusAccept、TYHomeStatusReject，未接受加入的家庭成员将无法使用该家庭下的设备等功能，拒绝加入家庭后将无法在获取家庭列表接口中获取到该家庭信息。

Objc:

```objc
- (void)initMemberList {
  //	_home = [WiserSmartHome homeWithHomeId:homeId];
  	[_home joinFamilyWithAccept:YES success:^(BOOL result) {
        NSLog(@"join success");
    } failure:^(NSError *error) {
        NSLog(@"join failure");
    }];
}
```



Swift:

```swift
func initMemberList(_ memberModel: WiserSmartHomeMemberModel) {
    home?.joinFamilyWithAccept(true, success: { (result: Bool) in
        print("join success")
    }, failure: { (error) in
        if let e = error {
            print("join failure: \(e)")
        }
    })
}
```



### 房间管理

单个房间信息管理相关的所有功能对应`WiserSmartRoom`类，需要使用roomId进行初始化。错误的roomId可能会导致初始化失败，返回`nil`。

#### 更新房间名字

Objc:

```objc
- (void)updateRoomName {
    [self.room updateRoomName:@"new_room_name" success:^{
        NSLog(@"update room name success");
    } failure:^(NSError *error) {
        NSLog(@"update room name failure: %@", error);
    }];
}
```

Swift:

```swift
func updateRoomName() {
    room?.updateName("new_room_name", success: {
        print("update room name success")
    }, failure: { (error) in
        if let e = error {
            print("update room name failure: \(e)")
        }
    })
}
```



#### 添加设备到房间

Objc:

```objc
- (void)addDevice {
    [self.room addDeviceWithDeviceId:@"devId" success:^{
        NSLog(@"add device to room success");
    } failure:^(NSError *error) {
        NSLog(@"add device to room failure: %@", error);
    }];
}
```

Swift:

```swift
func addDevice() {
    room?.addDevice(withDeviceId: "your_devId", success: {
        print("add device to room success")
    }, failure: { (error) in
        if let e = error {
            print("add device to room failure: \(e)")
        }
    })
}
```



#### 从房间中移除设备

Objc:

```objc
- (void)removeDevice {
    [self.room removeDeviceWithDeviceId:@"devId" success:^{
        NSLog(@"remove device from room success");
    } failure:^(NSError *error) {
        NSLog(@"remove device from room failure: %@", error);
    }];
}
```

Swift:

```swift
func removeDevice() {
    room?.removeDevice(withDeviceId: "your_devId", success: {
        print("remove device from room success")
    }, failure: { (error) in
        if let e = error {
            print("remove device from room failure: \(e)")
        }
    })
}
```



#### 添加群组到房间

Objc:

```objc
- (void)addGroup {
    [self.room addGroupWithGroupId:@"groupId" success:^{
        NSLog(@"add group to room success");
    } failure:^(NSError *error) {
        NSLog(@"add group to room failure: %@", error);
    }];
}
```

Swift:

```swift
func addGroup() {
    room?.addGroup(withGroupId: "your_groupId", success: {
        print("add group to room success")
    }, failure: { (error) in
        if let e = error {
            print("add group to room failure: \(e)")
        }
    })
}
```



#### 从房间中移除群组

Objc:

```objc
- (void)removeGroup {
    [self.room removeGroupWithGroupId:@"groupId" success:^{
        NSLog(@"remove group from room success");
    } failure:^(NSError *error) {
        NSLog(@"remove group from room failure: %@", error);
    }];
}
```

Swift:

```swift
func removeGroup() {
    room?.removeGroup(withDeviceId: "your_devId", success: {
        print("remove device from room success")
    }, failure: { (error) in
        if let e = error {
            print("remove device from room failure: \(e)")
        }
    })
}
```



#### 批量修改房间与群组、设备的关系

Objc:

```objc
- (void)saveBatchRoomRelation {
    [self.room saveBatchRoomRelationWithDeviceGroupList:(NSArray <NSString *> *)deviceGroupList success:^{
        NSLog(@"save batch room relation success");
    } failure:^(NSError *error) {
        NSLog(@"save batch room relation failure: %@", error);
    }];
}
```

Swift:

```swift
func saveBatchRoomRelation() {
    room?.saveBatchRoomRelation(withDeviceGroupList: [String]!, success: {
        print("save batch room relation success")
    }, failure: { (error) in
        if let e = error {
            print("save batch room relation failure: \(e)")
        }
    })
}
```

