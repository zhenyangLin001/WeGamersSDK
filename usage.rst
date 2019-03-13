################
Use SDK
################

IGGSDK 由多个功能模块组成（IGGSDK（Base）、IGGCRM等），以供游戏接入。

===========
Import SDK 
===========

1. 集成必要的功能模块

.. image:: _images/integration_sdk_step_1.png

- 上图红框标出了 **IGGSDK** 所有功能模块， **IGGCommonLibs** 为 **IGGSDK** 的基础模块，有接 **IGGSDK** 的游戏必须接入这个基础模块。
  **IGGDiagnosisTool** 是网络诊断工具模块，主要用于检查游戏当前运行的手机网络情况，如果不想使用这个功能，可以不集成到游戏里。
  **IGGSDK（Base）** 模块实现了IGG平台相关的基础功能，像登录、支付、分享、翻译等功能，游戏想接入IGG平台的登录等功能的话，就必须
  集成 **IGGSDK（Base）** 模块。**IGGCRM** 模块实现了客服功能，像玩家提交问题单，查看客服的回复，都已在这个模块里实现。
  所以想让游戏具有客服的功能，就可以集成 **IGGCRM** 模块，但是因为 **IGGCRM** 模块的一些基础服务依赖 **IGGSDK（Base）** 
  所以集成 **IGGCRM** 时，同时要集成 **IGGSDK（Base）** 模块。**IGGGeneralPayment** 模块作为集成国内常用的第三方支付的模块，
  一般有在国内运营的游戏都可以直接集成 **IGGGeneralPayment** 模块，从而直接使用这个模块的支付接口，快速完成支付开发。
  
2. 修改根工程编译脚本

- build.gradle文件修改：

.. image:: _images/integration_sdk_step_2.png

- 红框为修改的地方，**’../IGGCommonLibs/libs’** 为必修改项，其他修改根据游戏接入的实际情况来修改。

.. _rst-blank-lines:

- settings.gradle文件修改：

.. image:: _images/integration_sdk_step_3.png

- 红框为修改的地方，**’IGGCommonLibs’** 为必修改项，其他修改根据游戏接入的实际情况来修改。


3. 修改游戏应用工程的编译脚本文件

.. image:: _images/integration_sdk_step_4.png

- 红框为修改的地方，**’compile project(’:IGGCommonLibs’)** 为必修改项，其他修改根据游戏接入的实际情况来修改。

==================
Import SDK (Maven)
==================

1、在build.gradle添加IGGSDK仓库地址 

.. code-block:: xml

  allprojects {
      repositories {
          maven {
              url 'http://maven.skyunion.net/repository/IGGSDK-releases/'  // IGGSDK仓库地址
              name 'IGG'
          }
      }
  }


2、在build.gradle按需添加IGGSDK模块依赖

.. code-block:: xml

  dependencies {
      compile 'com.igg.sdk:Etc:$latest_version' // IGGSDK 基础库，主要包含一些 IGGSDK 依赖的第三方 SDK ，必加。
      
      compile 'com.igg.sdk:IGGSDK:$latest_version' // IGGSDK 库，集成有登录、支付、推送等功能，必加。
      compile 'com.igg.sdk:AmazonSDK:$latest_version' // Amazon 提供的支付相关库，接 Amazon 支付必加。
      compile 'com.igg.sdk:TrivialDrive:$latest_version' // Google 提供的支付相关库，接 Google 支付必加。
      compile 'com.igg.sdk:GeneralPayment:$latest_version' // 支付宝、微信支付相关库，接支付宝、微信支付必加。
      
      compile 'com.igg.sdk:CRM:$latest_version' // CRM相关库
      compile 'com.igg.sdk:PhotoSelector:$latest_version' // CRM 依赖的库，接CRM必加。
      compile 'com.igg.sdk:PullToRefresh:$latest_version' // CRM 依赖的库，接CRM必加。
      compile 'com.igg.sdk:ViewPagerIndicator:$latest_version' // CRM 依赖的库，接CRM必加。
      
      compile 'com.igg.sdk:DiagnosisTool:$latest_version' // 网络诊断相关库，有需要再加。
  ｝

注意: **latest_version变量值请设置为IGGSDK最新版本号，最新版本号查询站点：** http://sdk.skyunion.net

===========================
Permissions Configuration
===========================

SDK needs to read device information, provide GCM support, to gaurantee the normal operation of SDK, please refer to AndroidManifest.xml in IGGSDKDemo to configurate the project permissions.

===============
Initialize SDK
===============

1.API编译版本号需要在assets/appConfig.properties文件中设置（API 23 以上编译的会弹动态权限问题）

.. image:: _images/appconfig.png


2.This is mainly to complete the configuration in IGGSDK class. For details refer to LoginDemoActivity in IGGSDKDemo project.

.. code-block:: java

  IGGSDK iggsdk = IGGSDK.sharedInstance();
  // 设置游戏玩家信息，支付完提交给服务端使用
  GameDelegate  gameDelegate = new GameDelegate();
  iggsdk.setGameDelegate(gameDelegate);
  // 设置获取储存权限（增加随机数作为设备ID情况下使用）
  GamePermissionsDelegate gamePermissionsDelegate = new GamePermissionsDelegate();
  iggsdk.setDevicePermissionsDelegate(gamePermissionsDelegate);
  
  //会员接口（cgi）是否要开启安全请求。
  IGGSDK.sharedInstance().setUMSTransportSecurityEnabled(IGGSDKDemoConfigure.sharedInstance().isUMSTransportSecurityEnabled());
  
  // 用在跳转（客户功能，论坛等）, 游戏配置 config , CRM 的路由选择
  iggsdk.setRegionalCenter(IGGIDC.SND);
  // 数据中心设置， 用在收集日志, 录像功能的路由选择
  iggsdk.setDataCenter(IGGIDC.SND);
  // Set API Family also
  iggsdk.setFamily(IGGFamily.FP);
  // Game ID, provided by IGG
  iggsdk.setGameId("4001999902");
  // Secret Key, provided by IGG
  iggsdk.setSecretKey("8a6431c7d79002bf207102fe613e01a2"); 
  // Enhanced Secret Key, provided by IGG
  iggsdk.setEnhancedSecretKey("8a6431c7d79002bf207102fe613e01a2");
  // 不在国内运营设置为false 在国内运营的设置为true
  iggsdk.setChinaMainland(false);
  // 使用SD卡存储设备ID，请勿忘记需要的对应权限配置，false则表示不用SD卡
  iggsdk.setUseExternalStorage(true);
  // Payment Key, provided by Google Play
  iggsdk.setPaymentKey("MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAuLDWWE/M7RpGRLotqDcD1xOX1a72yFUpzzKNbxIb/490zPUpch9NEZOsJ2IeXRfWKtYXm+PLdZVuHZNz8m6IHD1oRG+zGzjMp4KkLMXzluxAsuaq6bgDibjnPQ8zQLXVp8HYySz7vr46i3lTxQXGf5MZw4XPlhIWN/RAgk5FBnClM1nWtKY/m6YnTe5S8Frfanfyh73e+2piig6EGHCngg80vQePkTYp36/4bnpBSWjWPMIOk9/auTEexerY9tt3JIMR9MGUI2xi0s8abq2iAk1t0pPtUoxxcDdVVzbUWwIabFVZHSTersJQwR+TwfuAEBlPFokQVyBPEmq0Y671+wIDAQAB");
  // GCM 配置
  iggsdk.setGCMSenderId("489219977954");
  // 收到推送消息是否自定义处理，false 为默认处理（该方式也可以通过重写方法达到自定义处理的效果）
  // 可以参考 GCMIntentDemoService 这个接收类
  iggsdk.setPushMessageCustomHandle(this.getApplication(), false);
  // IGGSDK 是否使用 https 连接, 不使用则设置为 false
  iggsdk.setSwitchHttps(false);
  // Application Context
  iggsdk.setApplication(this.getApplication());
  // 监听初始完
  iggsdk.initialize(new InitFinish());
  
  // 需要研发实现以下接口
  public class GameDelegate implements IGGGameDelegate {

    @Override
    public IGGCharacter getCharacter() {
        // 设置登陆者游戏的角色ID和角色名
        IGGCharacter character = new IGGCharacter();
        character.setCharId("1");
        character.setCharName("name");
        return character;
    }

    @Override
    public IGGServerInfo getServerInfo() {
        // 设置登陆者归属的服务器ID
        IGGServerInfo serverInfo = new IGGServerInfo();
        serverInfo.setServerId("1");
        return serverInfo;
    }
  }
  
    /**
     * IGGSDK初始完后执行
     */
    public class InitFinish implements IGGSDK.IGGSDKinitFinishListener {

        @Override
        public void finish() {
            // 初始完后执行研发的逻辑代码 runnable
            handler.postDelayed(runnable);
        }
    }
  
    private Runnable permissionRunnable;
    final private int REQUEST_CODE_ASK_MULTIPLE_PERMISSIONS = 10;
    // 实现获取权限过程
    public class GamePermissionsDelegate implements IGGDevicePermissionsDelegate {
        private int tmpSdkVersion;

        @Override
        @TargetApi(Build.VERSION_CODES.M)
        public void requestDeviceIdPermissions(Runnable runnable) {
            try {
                tmpSdkVersion = Build.VERSION.SDK_INT;
                Log.e(TAG, "tmpSdkVersion:" + tmpSdkVersion);
            } catch (NumberFormatException e) {
                tmpSdkVersion = 0;
            }

            if (tmpSdkVersion < 23) {
                return;
            }

            permissionRunnable = runnable;
            String[] permissions = {Manifest.permission.WRITE_EXTERNAL_STORAGE};
            LoginGameActivity.this.requestPermissions(permissions, REQUEST_CODE_ASK_MULTIPLE_PERMISSIONS);
        }
    }

    /**
     * 用户权限处理，碰到
     * 如果全部获取, 则直接过.
     * 如果权限缺失, 则提示Dialog.
     *
     * @param requestCode  请求码
     * @param permissions  权限
     * @param grantResults 结果
     */
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        if (requestCode == REQUEST_CODE_ASK_MULTIPLE_PERMISSIONS && hasAllPermissionsGranted(grantResults)) {
            // 获取权限，继续执行以下
            IGGSDK.sharedInstance().initialize(new InitFinish());
        } else {
            // 被拒绝，提示
            // showToast("no permissions");
            showMessageOK("Permission is Denied. The game was refused to enter",
                    new DialogInterface.OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                            finish();
                        }
                    });
        }
    }

    /**
    * 判断是否有权限
    *
    */
    private boolean hasAllPermissionsGranted(@NonNull int[] grantResults) {
        for (int grantResult : grantResults) {
            if (grantResult == PackageManager.PERMISSION_DENIED) {
                return false;
            }
        }

        return true;
    }
    