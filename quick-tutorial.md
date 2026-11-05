# 快速入门

（贴片/焊接完成后，安装好镜头座与镜头）
- 下载最新SD卡发布文件，并烧录到SD卡中：https://github.com/HumpbackLab/tidesense/releases/latest
- 使用USB线缆连接设备到电脑，此时应能识别一个RNDIS设备，如果未识别，参考附录。
- 下载代码：https://github.com/HumpbackLab/tidesense 部署开发环境（强烈建议使用podman/docker 并使用tidesense已发布的开发镜像，以确保环境一致）
- 编译构建TideConfigurator 并运行 https://github.com/HumpbackLab/tide_configurator 
- 使用网页访问TideConfigurator，连接设备: 192.168.123.99, 用户名：root, 密码：tidesense
- 在 相机标定 页面，选择镜头模式（鱼眼）， 点击启动预览，此时应该能出现双目的图像
- 根据预览图像拧动镜头来对焦，直到左右相机视野都足够清晰，使用固定圈固定此时镜头位置，防止松动
- 进入 VIO 部署页，点击启动VIO，此时设备上会基于SD上预标定的文件运行schurvins。（由于各个设备安装、对焦可能有所差距，所以此时若发散是正常的，需要各个设备自行标定一次）。确认网页上能观察到vio的轨迹即可


# 附录：Win10/11安装RNDIS驱动

## 手动指定 RNDIS 驱动
如果设备出现在“其他设备”中、带有黄色感叹号，或者被识别为未知 USB 设备，可手动使用
Windows 自带的 RNDIS 驱动：

1. 保持 TideSense 与电脑连接，按 `Win + X`，打开“设备管理器”。
2. 在“其他设备”或“网络适配器”中找到 TideSense 对应设备。无法确认时，可拔插 USB
   观察哪一项随之消失和出现。
3. 右键该设备，选择“更新驱动程序”。
4. 选择“浏览我的电脑以查找驱动程序”。
5. 选择“让我从计算机上的可用驱动程序列表中选取”。
6. 如果 Windows 要求选择设备类型，选择“网络适配器”。
7. 制造商选择“Microsoft”，型号选择“Remote NDIS Compatible Device”或
   “USB RNDIS Adapter”，然后点击“下一步”。
8. 如果出现兼容性警告，确认所选型号为 RNDIS 网络适配器后继续安装。
9. 安装完成后，设备应移动到“网络适配器”分类。拔插一次 USB，等待网络重新连接。