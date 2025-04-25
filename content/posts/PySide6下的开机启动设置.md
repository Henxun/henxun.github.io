---
title: PySide6的开机启动设置
tags:
  - pyside6
  - qt
  - 开机启动
summary: PySide6应用怎么实现在windows和macos上开机启动
date: 2025-04-25
---
## Windows开机启动设置
方法1：将快捷方式拷贝至“启动”目录
```
app_data_directory = QStandardPaths.writableLocation(QStandardPaths.StandardLocation.AppDataLocation)
startup_directory = QDir.cleanPath(app_data_directory + '/../Microsoft/Windows/Start Menu/Programs/Startup')

# 将快捷方式拷贝到启动目录
```
方法2：使用注册表
```
# 添加开机启动
# --autostart会作为启动参数传入，你也可以添加其它启动参数来实现启动时需要实现的操作
setting = QSetting('HKEY_CURRENT_USER\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run',  QSettings.Format.NativeFormat)
setting.setValue('Your app name', f'"Executable path" --autostarrt')
setting.sync()

# 取消开机启动
setting.remove('Your app name')
setting.sync()
```
## Mac OS开机启动设置
方法1：使用Apple Script
```
# 添加开机启动
# path设置为App路径
# hidden设置为true时，启动时会隐藏窗口
tell application "System Events" to
	make login item at end with properties {name: "Your app name", path: "Your app path", hidden: true}
end tell

# 取消开机启动
tell application "System Events"
	set theItems to login items whose path is "Your app path"
	repeat with aItem in theItems
		delete aItem
	end repeat
end tell
```
方法2：使用配置文件
```
# 在~/Library/LaunchAgents目录下创建.plist文件

path = QDir.cleanPath(QStandardPaths.writableLocation(QStandardPaths.StandardLocation.HomeLocation) + QDir.seperator() + 'Library/LaunchAgents/Your config.plist)
setting = QSettings(path, QSettings.Format.NativeFormat)
setting.setValue('Label', 'com.company.product.startup')
setting.setValue('RunAtLoad', True) # 关键设置
setting.setValue('Disabled', False)
setting.setValue('ProgramArguments', [ 'Executable path', '--autostart'])
setting.sync()
```