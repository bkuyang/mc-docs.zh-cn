---
author: conceptdev
ms.service: app-service-mobile
ms.topic: include
origin.date: 08/23/2018
ms.date: 08/23/2018
ms.author: v-tawe
ms.openlocfilehash: e3144df37c21ecf3639a2174986eb77820af50b1
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "71006552"
---
#### <a name="configure-the-ios-project-in-xamarin-studio"></a>在 Xamarin Studio 中配置 iOS 项目
1. 在 Xamarin.Studio 中，打开 **Info.plist**，并使用前面随新应用 ID 创建的捆绑 ID 来更新“捆绑标识符”  。

    ![](./media/app-service-mobile-xamarin-ios-configure-project/mobile-services-ios-push-21.png)
2. 向下滚动到“后台模式”  。 选择“启用后台模式”  框和“远程通知”  框。

    ![](./media/app-service-mobile-xamarin-ios-configure-project/mobile-services-ios-push-22.png)
3. 在解决方案面板中双击项目，打开“项目选项”  。
4. 在“生成”  下面选择“iOS 捆绑签名”  ，选择刚为此项目设置的标识和预配配置文件。

   ![](./media/app-service-mobile-xamarin-ios-configure-project/mobile-services-ios-push-20.png)

   这确保项目使用新配置文件进行代码签名。 有关正式的 Xamarin 设备预配文档，请参阅 [Xamarin 设备预配]。

#### <a name="configure-the-ios-project-in-visual-studio"></a>在 Visual Studio 配置 iOS 项目
1. 在 Visual Studio 中，右键单击项目，并单击“属性”。 
2. 在属性页中，单击“iOS 应用程序”选项卡，并使用先前创建的 ID 更新“标识符”。  

    ![](./media/app-service-mobile-xamarin-ios-configure-project/mobile-services-ios-push-23.png)
3. 在“iOS 捆绑签名”  选项卡中，选择刚为此项目设置的标识符和预配配置文件。

    ![](./media/app-service-mobile-xamarin-ios-configure-project/mobile-services-ios-push-24.png)

    这确保项目使用新配置文件进行代码签名。 有关正式的 Xamarin 设备预配文档，请参阅 [Xamarin 设备预配]。
4. 双击 Info.plist 打开它，并在“后台模式”下启用“远程通知”。

[Xamarin 设备预配]: http://developer.xamarin.com/guides/ios/getting_started/installation/device_provisioning/