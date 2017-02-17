---
title: Xcode自动打包脚本
date: 2017-01-09 15:29:51
tags: [Xcode, 打包脚本, Python]
categories: iOS
---

每次Xcode升级，都需要修改我们的自动打包脚本，这确实很头大。正好笔者也遇到了这个问题，下面就来仔细讲一下Xcode7和Xcode8之间自动编译并打包的差异。

#### Xcode7

##### 代码编译并归档

```python
xcodebuild -workspace "${SCHEME_NAME}.xcworkspace" -scheme "${SCHEME_NAME}"

-sdk iphoneos -configuration "${BUILD_CONFIGURATION}" build CODE_SIGN_IDENTITY="${IDENTITY_NAME}"

PROVISIONING_PROFILE="${PROVISIONING_PROFILE}" SYMROOT=${BUILD_PATH}
```

变量说明：

SCHEME_NAME：scheme名称

BUILD_CONFIGURATION：构建配置（比如inhouse/release/enterprise）

IDENTITY_NAME：证书的文件名（在keychain中可以获取到对应的文件名）

PROVISIONING_PROFILE：授权文件对应的文件名称（形式如：49cac5ce-3c77-472b-9abb-7c463a8bea92）

BUILD_PATH：编译之后存放的目录

##### 归档文件导出

```Python
xcrun -sdk "${TARGET_SDK}" -v PackageApplication "${PROJECT_BUILDDIR}/${SCHEME_NAME}.app" -o "${BUILD_OUTPUT_DIR}/${APP_NAME}.ipa"
```

变量说明：

TARGET_SDK：iphoneos（sdk版本，会根据系统当前sdk自动选择）

SCHEME_NAME：scheme名称

PROJECT_BUILDDIR：编译的目录

BUILD_OUTPUT_DIR：打包后的输出目录

APP_NAME：文件名称

------

#### Xcode8

代码编译并归档

```python
xcodebuild -scheme "${SCHEME_NAME}" -sdk "${TARGET_SDK}"

-archivePath "${PROJECT_BUILDDIR} ${SCHEME_NAME}.xcarchive"

-configuration Release PROVISIONING_PROFILE="${PROVISIONING_PROFILE}" archive
```

变量说明：

TARGET_SDK：iphoneos（sdk版本，会根据系统当前sdk自动选择）

SCHEME_NAME：scheme名称

PROJECT_BUILDDIR：编译的目录

PROVISIONING_PROFILE：授权文件对应的文件名称（形式如：49cac5ce-3c77-472b-9abb-7c463a8bea92）

##### 归档文件导出

```python
xcodebuild -exportArchive -archivePath "${PROJECT_BUILDDIR}/${SCHEME_NAME}.xcarchive"

-exportOptionsPlist"${EXPORT_PLIST}" -exportPath "${BUILD_OUTPUT_DIR}"
```

变量说明：

SCHEME_NAME：scheme名称

PROJECT_BUILDDIR：编译的目录

EXPORT_PLIST：plist文件路径（导出相关信息的配置）

BUILD_OUTPUT_DIR：ipa最终的导出目录

##### Plist格式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>method</key>
    <string>ad-hoc</string>
    <key>teamID</key>
    <string>xxxxxxx</string>
</dict>
</plist>
```

依次按照上面的步骤，写完编译脚本，感觉目标就要完成了。但是Xcode8的问题才刚刚开始

1. Xcode8多了一个Automatically manage signing（自动管理签名）

   解决：需要将自动管理替换为手动管理

   ```shell
   sed -i '' 's/ProvisioningStyle = Automatic;/ProvisioningStyle = Manual;/g' ${PROJECT_DIR}/project.pbxproj"
   ```

2. 改成手动管理后，由于我们的企业版和App Store的版的bundle id和开发者Team完全不一样

   解决：通过脚本修改Bundle ID，然后将工程中的Developer 替换为Distribution

   ```shell
   //替换bundle id

   sed -i '' 's/${OLD_BUNDLE_ID}/${NEW_BUNDLE_ID}/g' ${PROJECT_DIR}/project.pbxproj

   //替换打包类型

   sed -i '' 's/iPhone Developer/iPhone Distribution/g' ${PROJECT_DIR}/project.pbxproj
   ```

  最终的编译脚本已经放到了[github](https://github.com/edison0951/python_automate)，感兴趣的朋友可以自行下载查看