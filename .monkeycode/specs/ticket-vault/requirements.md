# 票密夹APP需求文档

## 引言

票密夹是一款面向个人用户的私密高效票据电子化管理APP，专注身份证、户口本、门票、车票、发票等各类票据的高清扫描、本地加密存储、智能整理与安全备份。核心原则是隐私可控、本地优先、操作极简、低资源占用，无强制云端上传、无自动AI调用、无隐私泄露风险。

### 目标平台
- 初期支持Android单端
- 后期可扩展至iOS

### 开发策略
- 第一阶段：基础框架与核心页面
- 第二阶段：本地OCR与票据管理
- 第三阶段：隐私票据与本地备份
- 第四阶段：云端同步与环境变量服务
- 第五阶段：AI功能与优化上线
- 第六阶段：后续迭代功能（账号体系、会员功能、WebDAV、短信登录等）

## 词汇表

- **票据(Ticket)**：用户通过拍照或上传方式电子化的实体票据，包含图片和元数据
- **隐私票据(Private Ticket)**：设置了隐私标识的票据，在列表中显示小锁标识，查看详情需输入密码验证
- **OCR识别**：光学字符识别，将票据图片中的文字转换为可编辑文本
- **合集(Collection)**：用户创建的票据分组，用于分类管理
- **S3适配层**：基于S3协议的数据库同步适配层，支持增量差分同步
- **BFF服务**：Backend-For-Frontend代理服务，负责接口转发、鉴权、限流
- **环境变量服务**：存储第三方API密钥的加密静态网站服务
- **到期日期(Due Date)**：票据的有效期或到期时间，不自动提醒用户
- **WebP**：Google开发的网页图片格式，用于票据图片压缩存储
- **账号体系(Account)**：用户身份验证系统，支持手机号/邮箱注册登录，为会员功能预留扩展接口
- **Hive数据库**：轻量级Flutter本地数据库，支持加密，Flutter 3.x兼容

## 需求

### 需求1：一站式票据新建

**用户故事：** AS 一个普通用户，我想要在一站式页面完成票据拍照、图片处理、信息录入与保存，以便快速高效地管理我的票据。

#### Acceptance Criteria

1. WHEN 用户点击「拍照」按钮 THEN 系统 SHALL 打开设备相机进行拍照
2. WHEN 用户点击「从相册上传」按钮 THEN 系统 SHALL 打开设备相册供用户选择图片
3. WHEN 图片加载完成 THEN 系统 SHALL 在图片处理区实时预览图片
4. WHEN 用户点击「识别文字」按钮 THEN 系统 SHALL 启动OCR识别并将结果显示在文本框中
5. IF 前一个OCR任务未完成 THEN 系统 SHALL 弹窗提示「当前识别任务未结束，请稍后再试」
6. WHEN OCR识别完成 THEN 系统 SHALL 自动填充识别结果至OCR文本框
7. WHEN 用户点击「自动扫描矫正」按钮 THEN 系统 SHALL 对图片进行自动透视矫正
8. WHEN 用户点击「边框裁剪」按钮 THEN 系统 SHALL 启用裁剪工具允许用户调整边界
9. WHEN 用户点击「自由角度旋转」按钮 THEN 系统 SHALL 启用旋转工具允许用户调整角度
10. WHEN 用户点击「缩放」按钮 THEN 系统 SHALL 启用缩放工具允许用户调整大小
11. WHEN 用户点击「清晰化」按钮 THEN 系统 SHALL 对图片进行清晰度增强处理
12. WHEN 用户点击「去阴影」按钮 THEN 系统 SHALL 对图片进行阴影去除处理
13. WHEN 用户点击「保存票据」按钮 THEN 系统 SHALL 保存票据至本地数据库并返回首页
14. WHERE 用户开启AI功能 AND WHERE OCR已完成 THEN 系统 SHALL 自动填充票据简介和推荐标签
15. WHERE 用户修改OCR文本 THEN 系统 SHALL NOT 自动重新触发AI整理
16. WHEN 用户开启「设为隐私票据」开关 THEN 系统 SHALL 保存时将票据标记为隐私票据

### 需求2：票据表单字段

**用户故事：** AS 一个普通用户，我想要填写完整的票据信息，以便更好地整理和检索我的票据。

#### Acceptance Criteria

1. THE 系统 SHALL 提供标题输入框允许用户手动填写票据名称
2. THE 系统 SHALL 提供OCR识别结果文本框允许用户查看和编辑识别内容
3. THE 系统 SHALL 提供票据简介文本框用于存储AI整理后的摘要描述
4. THE 系统 SHALL 提供标签输入框允许用户手动输入或选择AI推荐的标签
5. THE 系统 SHALL 提供合集选择下拉框允许用户选择已有合集或新建快速合集
6. THE 系统 SHALL 提供位置选择器允许用户手动输入位置或自动获取当前位置
7. WHERE 用户点击自动获取位置 THEN 系统 SHALL 尝试获取当前位置
8. IF 自动获取位置失败 THEN 系统 SHALL 允许用户手动输入位置信息
9. THE 系统 SHALL 提供日期选择器默认填充当前时间且允许用户修改
10. THE 系统 SHALL 提供到期日期选择器允许用户选择性地填写票据有效期
11. THE 系统 SHALL 对未填写到期日期的票据不进行任何形式的提醒
12. THE 系统 SHALL 提供「设为隐私票据」开关默认关闭

### 需求3：首页票据列表

**用户故事：** AS 一个普通用户，我想要在首页查看和管理我的所有票据，以便快速找到需要的票据。

#### Acceptance Criteria

1. THE 系统 SHALL 在首页顶部显示「新建票据」按钮
2. THE 系统 SHALL 提供下拉筛选菜单允许用户按标签或日期筛选票据
3. THE 系统 SHALL 提供全局搜索框允许用户搜索票据标题和OCR内容
4. WHERE 用户未解锁隐私票据 THEN 系统 SHALL 仅搜索普通票据
5. WHERE 用户已解锁隐私票据 THEN 系统 SHALL 提供选项允许用户选择是否包含隐私票据
6. THE 系统 SHALL 提供排序切换按钮允许用户按时间升序/降序或按名称排序
7. THE 系统 SHALL 采用分页懒加载方式展示票据列表每页10条
8. THE 系统 SHALL 在票据卡片中显示缩略图、标题、日期、标签和隐私标识
9. WHEN 用户下拉刷新 THEN 系统 SHALL 刷新票据列表
10. WHEN 用户上拉加载 THEN 系统 SHALL 加载更多票据
11. WHEN 用户点击票据卡片 THEN 系统 SHALL 导航至票据详情页
12. WHEN 用户长按任意票据卡片 THEN 系统 SHALL 进入多选模式

### 需求4：批量操作功能

**用户故事：** AS 一个普通用户，我想要对多个票据进行批量操作，以便高效管理我的票据。

#### Acceptance Criteria

1. WHEN 用户长按票据卡片 THEN 系统 SHALL 进入多选模式并在顶部显示操作栏
2. THE 系统 SHALL 在操作栏中仅显示「批量设为隐私」和「批量删除」两个功能按钮
3. WHEN 用户点击「批量设为隐私」 THEN 系统 SHALL 弹窗要求用户二次确认
4. WHEN 用户点击「批量删除」 THEN 系统 SHALL 弹窗要求用户二次确认
5. WHEN 用户完成操作 OR 点击取消 THEN 系统 SHALL 退出多选模式
6. THE 系统 SHALL 异步执行批量操作不阻塞UI

### 需求5：票据详情页

**用户故事：** AS 一个普通用户，我想要查看票据的完整信息和高清图片，以便详细了解票据内容。

#### Acceptance Criteria

1. THE 系统 SHALL 在详情页顶部显示返回按钮、编辑按钮和更多操作按钮
2. THE 系统 SHALL 在更多操作中提供导出功能支持图片和PDF格式
3. THE 系统 SHALL 在更多操作中提供删除功能
4. THE 系统 SHALL 在更多操作中提供设为隐私/取消隐私功能
5. THE 系统 SHALL 显示票据高清图片和完整元数据信息
6. THE 系统 SHALL 显示的元数据包括标题、OCR结果、简介、标签、合集、位置、创建日期、到期日期和存储位置

### 需求6：合集管理功能

**用户故事：** AS 一个普通用户，我想要创建和管理票据合集，以便分类整理我的票据。

#### Acceptance Criteria

1. WHEN 用户从侧边抽屉菜单点击「合集管理」 THEN 系统 SHALL 导航至合集管理页面
2. THE 系统 SHALL 提供新建合集功能
3. THE 系统 SHALL 提供编辑合集名称和封面的功能
4. THE 系统 SHALL 提供删除合集功能
5. WHEN 用户删除合集 THEN 系统 SHALL 弹窗要求用户选择处理方式
6. THE 系统 SHALL 提供三种删除合集的处理方式
7. WHERE 用户选择「票据移至未分类」 THEN 系统 SHALL 将合集中的票据移至未分类合集
8. WHERE 用户选择「删除合集中所有票据」 THEN 系统 SHALL 删除合集中的所有票据并要求二次确认
9. WHERE 用户选择「仅删除合集」 THEN 系统 SHALL 仅删除合集将票据的合集字段置空
10. THE 系统 SHALL 提供查看合集中票据列表的功能
11. THE 系统 SHALL 在首页筛选菜单中提供按合集筛选票据的功能

### 需求7：隐私票据功能

**用户故事：** AS 一个注重隐私的用户，我想要将敏感票据设置为隐私票据，以便安全地管理我的私密票据。

#### Acceptance Criteria

1. THE 系统 SHALL 支持用户将任意票据设置为隐私票据
2. THE 系统 SHALL 在票据列表中以小锁标识显示隐私票据
3. THE 系统 SHALL 支持6位数字密码和复杂密码两种方式设置隐私验证
4. THE 系统 SHALL 将密码哈希存储于系统安全硬件
5. THE 系统 SHALL 采用独立存储目录方案存储隐私票据数据
6. THE 系统 SHALL 使用AES-256-GCM逐文件加密隐私票据图片
7. THE 系统 SHALL 将加密密钥仅存储于系统安全硬件
8. WHEN 用户查看隐私票据详情 THEN 系统 SHALL 要求用户输入密码或使用生物识别验证
9. THE 系统 SHALL 在用户锁屏或退出APP后自动销毁内存中的验证状态
10. THE 系统 SHALL 默认禁止云端同步隐私票据数据
11. WHERE 用户手动开启隐私票据云同步 THEN 系统 SHALL 仅上传加密密文
12. THE 系统 SHALL 提供密码提示功能
13. WHERE 用户忘记密码 THEN 系统 SHALL 提供通过密码或验证码验证后重新设置新密码的选项
14. THE 隐私票据列表 SHALL 与首页保持一致支持筛选、搜索和排序

### 需求8：设置页面

**用户故事：** AS 一个普通用户，我想要在设置页面管理APP的各项功能开关和配置，以便定制化我的使用体验。

#### Acceptance Criteria

1. THE 系统 SHALL 提供账号安全设置分组包含账号注册、登录、密码修改功能
2. THE 系统 SHALL 提供存储与备份设置分组包含手动备份和恢复操作按钮
3. THE 系统 SHALL 提供识别设置分组包含云端OCR总开关默认关闭
4. THE 系统 SHALL 提供AI功能设置分组包含AI整理、标签推荐、位置获取开关默认关闭
5. THE 系统 SHALL 提供隐私票据设置分组包含密码修改和密码提示功能
6. THE 系统 SHALL 提供通用设置分组
7. THE 系统 SHALL 确保所有扩展功能开关默认关闭
8. THE 系统 SHALL 为会员功能预留扩展接口

### 需求9：本地OCR识别

**用户故事：** AS 一个普通用户，我想要将票据图片中的文字识别出来，以便快速提取票据信息。

#### Acceptance Criteria

1. THE 系统 SHALL 集成Google ML Kit Text Recognition v2离线OCR引擎
2. THE 系统 SHALL 采用动态模型下载方案首次使用时下载中文识别模型
3. THE 系统 SHALL 确保OCR初始安装体积增加约3MB
4. THE 系统 SHALL 确保识别速度小于1.5秒
5. THE 系统 SHALL 确保内存占用小于30MB
6. THE 系统 SHALL 确保中文识别率大于等于96%
7. THE 系统 SHALL 采用串行任务控制防止并发资源占用

### 需求10：图片处理功能

**用户故事：** AS 一个普通用户，我想要对票据图片进行处理，以便获得更清晰的扫描效果。

#### Acceptance Criteria

1. THE 系统 SHALL 集成OpenCV Flutter插件进行图片处理
2. THE 系统 SHALL 提供自动扫描矫正功能进行透视矫正
3. THE 系统 SHALL 提供边框裁剪功能允许用户调整边界
4. THE 系统 SHALL 提供自由角度旋转功能允许用户调整角度
5. THE 系统 SHALL 提供缩放功能允许用户调整大小
6. THE 系统 SHALL 提供清晰化功能进行边缘增强
7. THE 系统 SHALL 提供去阴影功能去除图片阴影
8. THE 系统 SHALL 确保所有处理操作实时预览
9. THE 系统 SHALL 通过异步线程处理避免UI阻塞
10. THE 系统 SHALL 确保OpenCV插件体积优化后约10-15MB

### 需求11：本地存储功能

**用户故事：** AS 一个普通用户，我想要将票据安全地存储在本地，以便离线访问我的票据。

#### Acceptance Criteria

1. THE 系统 SHALL 使用Hive数据库存储结构化数据
2. THE 系统 SHALL 在应用沙盒中分目录存储普通票据和隐私票据
3. THE 系统 SHALL 使用WebP格式压缩存储票据图片
4. THE 系统 SHALL 对隐私票据数据强制加密存储
5. THE 系统 SHALL 自动清理30天前未访问的缩略图缓存
6. THE 系统 SHALL 默认将图片压缩至1080P分辨率
7. THE 系统 SHALL 允许用户手动切换至720P分辨率

### 需求12：本地备份与恢复

**用户故事：** AS 一个普通用户，我想要备份和恢复我的票据数据，以便防止数据丢失。

#### Acceptance Criteria

1. THE 系统 SHALL 支持手动备份功能用户可在设置页一键触发
2. THE 系统 SHALL 将备份数据保存为AES-256加密ZIP压缩包文件后缀为.pmi.enc
3. THE 系统 SHALL 要求用户在备份时设置自定义密码
4. WHEN 用户选择恢复备份 THEN 系统 SHALL 要求用户输入密码
5. WHEN 用户确认恢复 THEN 系统 SHALL 覆盖本地现有数据前要求二次确认
6. THE 备份内容 SHALL 包含Hive数据库文件、票据图片、隐私票据加密数据和用户配置

### 需求13：云端同步功能

**用户故事：** AS 有多设备需求的用户，我想要将票据同步到云端，以便在不同设备间访问我的票据。

#### Acceptance Criteria

1. THE 系统 SHALL 默认关闭云端同步功能
2. THE 系统 SHALL 采用增量差分同步机制仅同步未同步、修改或删除的数据
3. THE 系统 SHALL 支持图片断点续传与差分上传
4. WHERE 设备连接Wi-Fi THEN 系统 SHALL 自动执行同步
5. WHERE 设备使用移动数据 THEN 系统 SHALL 需用户手动确认后同步
6. THE 系统 SHALL 实时展示同步状态包括未同步、同步中、成功、失败
7. WHERE 本地版本高于云端版本 THEN 系统 SHALL 自动覆盖云端数据
8. WHERE 本地版本低于云端版本 THEN 系统 SHALL 弹窗提示用户选择
9. WHERE 服务器故障 THEN 系统 SHALL 自动切换至离线模式
10. THE 云端存储 SHALL 仅保存加密后的密文
11. THE 云端存储 SHALL 默认禁止同步隐私票据数据

### 需求14：S3同步适配层

**用户故事：** AS 一个关注数据迁移的用户，我想要确保我的数据可以无缝迁移，以便终身无需重构系统。

#### Acceptance Criteria

1. THE 系统 SHALL 实现独立S3同步适配层
2. THE 系统 SHALL 通过适配层实现S3协议兼容
3. THE 系统 SHALL 确保本地数据结构与云端数据结构双向对齐
4. THE 系统 SHALL 支持增量同步机制

### 需求15：环境变量与BFF服务

**用户故事：** AS 一个关注数据安全的用户，我想要安全地管理第三方API密钥，以便防止密钥泄露。

#### Acceptance Criteria

1. THE 系统 SHALL 部署1Panel静态网站存储讯飞OCR和大模型密钥
2. THE 系统 SHALL 采用HTTP基础认证和HTTPS加密传输
3. THE 系统 SHALL 通过加密请求让APP获取密钥不落地存储
4. THE BFF服务 SHALL 采用Node.js+Express轻量化实现
5. THE BFF服务 SHALL 仅负责接口转发、鉴权和限流
6. THE BFF服务 SHALL 确保内存占用小于50MB

### 需求16：权限管理

**用户故事：** AS 一个注重隐私的用户，我想要控制APP的权限授权，以便保护我的隐私安全。

#### Acceptance Criteria

1. THE 系统 SHALL 仅申请相机权限用于票据拍照
2. THE 系统 SHALL 仅申请存储权限用于图片存取
3. THE 系统 SHALL 仅申请位置权限用于自动填充位置且默认关闭
4. THE 系统 SHALL 仅申请生物识别权限用于隐私票据验证且默认关闭
5. WHERE 用户拒绝授权 THEN 系统 SHALL 确保核心功能不受影响

### 需求17：性能优化

**用户故事：** AS 一个普通用户，我想要APP运行流畅响应迅速，以便获得良好的使用体验。

#### Acceptance Criteria

1. THE 系统 SHALL 确保安装包体积小于25MB
2. THE 系统 SHALL 确保APP启动时间小于2.5秒
3. THE 系统 SHALL 确保运行时帧率大于等于60fps
4. THE 系统 SHALL 采用分页懒加载避免一次性加载大量数据
5. THE 系统 SHALL 对图片进行按需加载和缓存限制最大缓存100张缩略图
6. THE 系统 SHALL 通过异步线程执行耗时操作
7. THE 系统 SHALL 确保OCR任务串行执行防止内存溢出
8. THE 系统 SHALL 在大图查看后自动释放内存

### 需求18：深色/浅色主题

**用户故事：** AS 一个普通用户，我想要APP支持深色和浅色主题切换，以便在不同环境下舒适使用。

#### Acceptance Criteria

1. THE 系统 SHALL 支持跟随系统主题自动切换
2. THE 系统 SHALL 在设置中提供主题选择选项
3. THE 系统 SHALL 确保深色主题下界面元素清晰可见

### 需求19：侧边抽屉菜单

**用户故事：** AS 一个普通用户，我想要通过侧边菜单快速导航到各个功能页面。

#### Acceptance Criteria

1. THE 系统 SHALL 在顶部导航栏右侧显示三点垂直菜单按钮
2. WHEN 用户点击菜单按钮 THEN 系统 SHALL 呼出侧边抽屉菜单
3. THE 侧边抽屉菜单 SHALL 依次显示首页、合集管理、隐私票据、设置、帮助与反馈
4. THE 侧边抽屉菜单 SHALL 在底部标注版本号
5. THE 侧边抽屉菜单 SHALL 采用极简列表设计无冗余元素

### 需求20：导航栏设计

**用户故事：** AS 一个普通用户，我想要通过导航栏快速访问核心功能和返回首页。

#### Acceptance Criteria

1. THE 系统 SHALL 在顶部固定导航栏左侧显示圆形用户头像
2. WHEN 用户点击头像 THEN 系统 SHALL 导航至个人中心
3. THE 系统 SHALL 在导航栏中间显示APP LOGO和应用名称
4. WHEN 用户点击LOGO或应用名称 THEN 系统 SHALL 一键返回首页

### 需求21：账号体系

**用户故事：** AS 一个普通用户，我想要创建账号并登录APP，以便安全地管理我的票据并在多设备间同步。

#### Acceptance Criteria

1. THE 系统 SHALL 支持用户通过手机号注册账号
2. THE 系统 SHALL 支持用户通过邮箱注册账号
3. THE 系统 SHALL 支持用户通过手机号或邮箱登录账号
4. THE 系统 SHALL 支持密码找回功能通过验证码验证
5. THE 系统 SHALL 支持设置和修改账号密码
6. THE 系统 SHALL 支持登录设备管理查看当前登录设备
7. THE 系统 SHALL 支持异常登录时强制下线
8. THE 系统 SHALL 为会员功能预留扩展接口支持后续升级
9. THE 会员功能 SHALL 区分免费用户和付费会员用户
10. THE 会员功能 SHALL 支持按月或按年订阅

### 需求22：会员权益

**用户故事：** AS 一个付费会员，我想要享受专属权益，以便获得更好的使用体验。

#### Acceptance Criteria

1. THE 会员功能 SHALL 支持高级云同步空间（免费用户2GB，会员50GB）
2. THE 会员功能 SHALL 支持云端OCR无限使用（免费用户每日10次）
3. THE 会员功能 SHALL 支持AI整理无限使用（免费用户每日5次）
4. THE 会员功能 SHALL 支持批量导出PDF功能
5. THE 会员功能 SHALL 支持优先客服支持
