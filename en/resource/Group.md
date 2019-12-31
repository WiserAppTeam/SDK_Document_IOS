## Group management

SDK supports the group management system. User can create group, change group name, manage devices of group, manage multiple devices via the group and dismiss group.

All functions of group are realized by using the `WiserSmartGroup` class, and all functions need to be initiated by using the group Id. Wrong group Id may cause initiation failure, and the `nil` will be returned.


### Create a group

Objc:

```objc
- (void)createNewGroup {
    
    [WiserSmartGroup createGroupWithName:@"your_group_name" productId:@"your_group_product_id" homeId:homeId devIdList:(NSArray<NSString *> *)selectedDevIdList success:^(WiserSmartGroup *group) {
        NSLog(@"create new group success %@:", group); 
    } failure:^(NSError *error) {
        NSLog(@"create new group failure");
    }];
}
```

Swift:

```swift
func createNewGroup() {
    WiserSmartGroup.createGroup(withName: "your_group_name", productId: "your_group_product_id", homeId: homeId, devIdList: ["selectedDevIdList"], success: { (group) in
        print("create new group success: \(group)")
    }) { (error) in
        print("create new group failure")
    }
}
```

### Obtain device list of group

Obtain the device list of product when the group is not created.

Objc:

```objc
- (void)getGroupDevList {
    
    [WiserSmartGroup getDevList:@"your_group_product_id" homeId:homeId success:^(NSArray<WiserSmartGroupDevListModel *> *list) {
        NSLog(@"get group dev list success %@:", list); 
    } failure:^(NSError *error) {
        NSLog(@"get group dev list failure");
    }];
}
```

Swift:

```swift
func getGroupDevList() {
    WiserSmartGroup.getDevList("your_group_product_id", homeId: 1, success: { (list) in
        print("get group dev list success \(list)")
    }) { (error) in
        print("get group dev list failure")
    }
}
```

Obtain the device list of a group when the group is created.

Objc:

```objc
- (void)getGroupDevList {
//    self.smartGroup = [WiserSmartGroup groupWithGroupId:@"your_group_id"];
    [self.smartGroup getDevList:@"your_group_product_id" success:^(NSArray<WiserSmartGroupDevListModel *> *list) {
        NSLog(@"get group dev list success %@:", list); 
    } failure:^(NSError *error) {
        NSLog(@"get group dev list failure");
    }];
}
```

Swift:

```swift
func getGroupDevList() {
    smartGroup?.getDevList("your_group_product_id", success: { (list) in
        print("get group dev list success \(list)")
    }, failure: { (error) in
        print("get group dev list failure")
    })
}
```

### Send dp command of a group

Objc:

```objc
- (void)publishDps {
//    self.smartGroup = [WiserSmartGroup groupWithGroupId:@"your_group_id"];
	
	NSDictionary *dps = @{@"1": @(YES)};
	[self.smartGroup publishDps:dps success:^{
		NSLog(@"publishDps success");
	} failure:^(NSError *error) {
		NSLog(@"publishDps failure: %@", error);
	}];
}
```
Swift:

```swift
func publishDps() {
    let dps = ["1" : true]
    smartGroup?.publishDps(dps, success: {
        print("publishDps success")
    }, failure: { (error) in
        print("publishDps failure")
    })
}
```

### Modify the group name

Objc:

```objc
- (void)updateGroupName {
//    self.smartGroup = [WiserSmartGroup groupWithGroupId:@"your_group_id"];

    [self.smartGroup updateGroupName:@"your_group_name" success:^{
        NSLog(@"update group name success");
    } failure:^(NSError *error) {
        NSLog(@"update group name failure: %@", error);
    }];
}
```
Swift:

```swift
func updateGroupName() {
    smartGroup?.updateName("your_group_name", success: {
        print("updateGroupName success")
    }, failure: { (error) in
        if let e = error {
            print("updateGroupName failure: \(e)")
        }
    })
}
```

### Modify group device list

Objc:

```objc
- (void)updateGroupRelations {
//    self.smartGroup = [WiserSmartGroup groupWithGroupId:@"your_group_id"];

    [self.smartGroup updateGroupRelations:(NSArray<NSString *> *)selectedDevIdList success:^ {
        NSLog(@"update group relations success");
    } failure:^(NSError *error) {
        NSLog(@"update group relations failure: %@", error);
    }];
}
```
Swift:

```swift
func updateGroupRelations() {
    smartGroup?.updateRelations(["selectedDevIdList"], success: {
        print("update group relations success")
    }, failure: { (error) in
        if let e = error {
            print("update group relations failure: \(e)")
        }
    })
}
```
### Dismiss group

Objc:

```objc
- (void)dismissGroup {
//    self.smartGroup = [WiserSmartGroup groupWithGroupId:@"your_group_id"];

    [self.smartGroup dismissGroup:^{
      NSLog(@"dismiss group success");
    } failure:^(NSError *error) {
      NSLog(@"dismiss group failure: %@", error);
    }];
}
```

Swift:

```swift
func dismissGroup() {
    smartGroup?.dismiss({
        print("dismiss group success")
    }, failure: { (error) in
        if let e = error {
            print("dismiss group failure: \(e)")
        }
    })
}
```


### Callback interface

Callback and data updating after the group sends DP.

Objc:

```objc

#pragma mark - WiserSmartGroupDelegate

- (void)group:(WiserSmartGroup *)group dpsUpdate:(NSDictionary *)dps {
  // TODO: update group panel UI here
}

```

Swift:

```swift
// MARK: WiserSmartGroupDelegate
func group(_ group: WiserSmartGroup!, dpsUpdate dps: [AnyHashable : Any]!) {
  // TODO: update group panel UI here
}
```

### Zigbee Group

The ZigBee  group requires that all devices belong to the same gateway and cannot cross the gateway.

#### Create Zigbee groups

Objc:

```objc
- (void)createNewGroup {
    
    [WiserSmartGroup createGroupWithName:@"your_group_name" homeId:homeID gwId:@"gwId" productId:@"your_group_product_id" success:^(WiserSmartGroup *group) {
        NSLog(@"create new group success %@:", group); 
    } failure:^(NSError *error) {
        NSLog(@"create new group failure");
    }];
}
```

Swift:

```swift
func createNewGroup() {
    WiserSmartGroup.createGroup(withName: "your_group_name", homeId: homeId, gwId: "gwId" productId: "your_group_product_id", success: { (group) in
        print("create new group success: \(group)")
    }) { (error) in
        print("create new group failure")
    }
}
```



#### Get a list of devices supported by the ZigBee group

Objc:

```objc
- (void)getGroupDevList {
    
    [WiserSmartGroup getDevListWithProductId:@"your_group_product_id" gwId:@"gwId" homeId:homeId success:^(NSArray<WiserSmartGroupDevListModel *> *list) {
        NSLog(@"get group dev list success %@:", list); 
    } failure:^(NSError *error) {
        NSLog(@"get group dev list failure");
    }];
}
```

Swift:

```swift
func getGroupDevList() {
    WiserSmartGroup.getDevList(withProductId: "your_group_product_id", gwId: "gwId", homeId: hoemId, success: { (list) in
        print("get group dev list success \(list)")
    }) { (error) in
        print("get group dev list failure")
    }
}
```


#### Add devices to the ZigBee group

Objc:

```objc
- (void)addDevice {
//    self.smartGroup = [WiserSmartGroup groupWithGroupId:@"your_group_id"];
    [self.smartGroup addZigbeeDeviceWithNodeList:@[@"nodeId1", @"nodeId2"]  success:^() {
        NSLog(@"get group dev list success %@:", list); 
    } failure:^(NSError *error) {
        NSLog(@"get group dev list failure");
    }];
}
```

Swift:

```swift
func addDevice() {
    martGroup.addZigbeeDevice(withNodeList: ["nodeId1", "nodeId2"], success: {
        print("add device success")
    }) { (error) in
        print("add device failure")
    }
}
```




#### Remove devices from ZigBee groups

Objc:

```objc
- (void)removeDevice {
    
    [self.smartGroup removeZigbeeDeviceWithNodeList:@[@"nodeId1", @"nodeId2"]  success:^() {
        NSLog(@"get group dev list success %@:", list); 
    } failure:^(NSError *error) {
        NSLog(@"get group dev list failure");
    }];
}
```

Swift:

```swift
func removeDevice() {
    martGroup.addZigbeeDevice(withNodeList: ["nodeId1", "nodeId2"], success: {
        print("remove device success")
    }) { (error) in
        print("remove device failure")
    }
}
```

#### Callback interface

Response of ZigBee devices to join or remove gateway groups

errorCode ：

- 1:Over the Scene Number Upper Limit
- 2: Subdevice timeout
- 3: Setting values beyond range
- 4: Writing errors
- 5: Other mistakes

Objc:

```objc

#pragma mark - WiserSmartGroupDelegate

- (void)group:(WiserSmartGroup *)group addResponseCode:(NSArray <NSNumber *>*)responseCode {
	// Responses of ZigBee devices joining groups
}

- (void)group:(WiserSmartGroup *)group removeResponseCode:(NSArray <NSNumber *>*)responseCode {
	// sub devices joined the gateway group successfully
}

```

Swift:

```swift
// MARK: WiserSmartGroupDelegate
func group(_ group: WiserSmartGroup?, addResponseCode responseCode: [NSNumber]?) {
    // Responses of ZigBee devices joining groups
}

func group(_ group: WiserSmartGroup?, removeResponseCode responseCode: [NSNumber]?) {
    // sub devices joined the gateway group successfully
}
```
