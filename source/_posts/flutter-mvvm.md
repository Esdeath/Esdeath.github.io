---
title: Flutter中使用Provider实现MVVM架构
date: 2020-08-29 09:10:14
tags:
---

# MVVM介绍

MVVM架构分为M(Model)、V(View)、VM(ViewModel)三个部分，他们分别处理自己的分工，在View和Model之间使用ViewModel作为中介者，使View和Model不受业务逻辑影响。

**Model**: 模型层，处理Api数据、模型相关业务

**View**: 视图层，UI呈现、使用者互动等。

**ViewModel**: 视图模型，处理逻辑、将数据绑定给View展示。

**Controller**: 负责主要事情就是将View和ViewModel进行绑定，生命周期管理

MVVM的核心思想即：通过ViewModel在View和Model之间建立一个连接，实现View和Model的双向绑定。

---

# Flutter中的MVVM应用

Flutter中通过`Provider`进行状态管理，来实现MVVM。在MVVM的原本的三个部分添加Services层，方便业务的分离.

Services: 是获取数据接口（服务器Server 或者 本地Store）

Model: 主要负责转换模型

View: 主要负责呈现UI，通过ViewModel获取数据并展示

ViewModel: 处理业务逻辑，将数据绑定给View展示

Controller:一般是stf，负责主要事情就是将View和ViewModel进行绑定，生命周期管理

---


## 1.定义模型
将网络请求回来的数据转换为对应的模型
```Dart 
class DeviceOnlineStatusData {
  String deviceId;
  String deviceType;
  String loginTime;
  String createdAt;

  DeviceOnlineStatusData({this.deviceId, this.deviceType, this.loginTime, this.createdAt});

  DeviceOnlineStatusData.fromJson(Map<String, dynamic> json) {
    deviceId = json['device_id'];
    deviceType = json['device_type'];
    loginTime = json['login_time'];
    createdAt = json['created_at'];
  }

  Map<String, dynamic> toJson() {
    final Map<String, dynamic> data = new Map<String, dynamic>();
    data['device_id'] = this.deviceId;
    data['device_type'] = this.deviceType;
    data['login_time'] = this.loginTime;
    data['created_at'] = this.createdAt;
    return data;
  }
}

```

## 2.定义ViewModel
这个ViewModel主要负责把请求回来的数据进行处理，并通知View层更新数据

```Dart
class DeviceOnlineStatusVM extends ChangeNotifier {
  bool isPcLogin = false;
  bool isPadLogin = false;

  String pcDeviceID = '';
  String pcDeviceType = '';

  String padDeviceID = '';
  String padDeviceType = '';

  setOnlineStatus(List<DeviceOnlineStatusData> statusData) {
    init();
    isPcLogin = true;
    pcDeviceID = item.deviceId;
    pcDeviceType = item.deviceType;
    isPadLogin = true;
    padDeviceID = item.deviceId;
    padDeviceType = item.deviceType;
    notifyListeners();
  }

  void init() {
    isPcLogin = false;
    isPadLogin = false;
    pcDeviceID = '';
    pcDeviceType = '';
    padDeviceID = '';
    padDeviceType = '';
  }
}
```

## 3.定义网络请求类
网络请求将请求回来的数据转换为模型，并更新ViewModel数据。通过extension进行横向拓展

```Dart
extension DeviceOnlineStatusServices on DeviceOnlineStatusVM {
  //获取设备状态
  Future<void> getOnlineStatusServices() async {
    HttpResp httpResp = await HttpApi().getOnlineStatus();
    if (httpResp.code == 0) {
      List jsonList = httpResp.value;
      List<DeviceOnlineStatusData> statusData = jsonList.map((e) => DeviceOnlineStatusData.fromJson(e)).toList();
      setOnlineStatus(statusData);
    }
  }

  //更新设备状态
  Future<void> setOnlineStatusServices(String deviceID, String deviceType) async {
    HttpResp httpResp = await HttpApi().postOnlineStatus(deviceID, deviceType);
    if (httpResp.code == 0) {
      List jsonList = httpResp.value;
      List<DeviceOnlineStatusData> statusData = jsonList.map((e) => DeviceOnlineStatusData.fromJson(e)).toList();
      setOnlineStatus(statusData);
    }
  }
}

```


## 4.View数据展示层
单独抽出View层，View用来渲染数据
```Dart

class DeviceOnlineStatusView extends StatelessWidget {
  const DeviceOnlineStatusView({Key key, this.viewModel}) : super(key: key);
  final DeviceOnlineStatusVM viewModel;
  @override
  Widget build(BuildContext context) {
    return Flex(
      direction: Axis.vertical,
      children: <Widget>[
         Column(
            crossAxisAlignment: CrossAxisAlignment.center,
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: <Widget>[Image.asset("assets/images/im_device_pc.png", width: 178, height: 92)],
              ),            
              const SizedBox(height: 14.0),
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: <Widget>[
                    TextImageButtonWidget(
                      text: "退出",
                      normalImage: "im_device_back.png",
                      disableImage: "im_device_back_disable.png",
                      buttonEnable: viewModel.isPcLogin,
                      onTapCallBack: () {
                        Utils.showCommonDialog(context: context, content: '是否退出电脑登录?').then((value) async {
                          if (value) {
                            viewModel
                                .setOnlineStatusServices(viewModel.pcDeviceID, viewModel.padDeviceType)
                                .then((value) {
                              if (!viewModel.isPadLogin && !viewModel.isPcLogin) {
                                Utils.popPage();
                              }
                            });
                          }
                        });
                      }),
                ],
              ),
            ],
          ),       
      ],
    );
  }
}

```

## 5.Controller用来将viewModel绑定View，并且管理生命周期

```Dart
class DeviceOnlineStatusPage extends StatefulWidget {
  const DeviceOnlineStatusPage({
    Key key,
    this.title,
    this.deviceOnlineStatusVM,
  }) : super(key: key);

  final String title;

  final DeviceOnlineStatusVM deviceOnlineStatusVM;

  @override
  State<StatefulWidget> createState() => _DeviceOnlineStatusPage();
}

class _DeviceOnlineStatusPage extends State<DeviceOnlineStatusPage> {
  @override
  void initState() {
    // 获取接口数据
    widget.deviceOnlineStatusVM.getOnlineStatusServices();

    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: NavigationAppBar(title: widget.title),
      body: ChangeNotifierProvider.value(
        value: widget.deviceOnlineStatusVM,
        child: Consumer<DeviceOnlineStatusVM>(
          builder: (context, _viewModel, _) {
            return DeviceOnlineStatusView(viewModel: _viewModel);
          },
        ),
      ),
    );
  }
}

```