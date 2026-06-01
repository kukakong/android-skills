# Android Skills - Code Wiki

> **项目版本**: Latest  
> **生成日期**: 2026-06-01  
> **许可证**: Apache License 2.0

---

## 目录

1. [项目概述](#项目概述)
2. [整体架构](#整体架构)
3. [主要模块职责](#主要模块职责)
4. [关键类与函数说明](#关键类与函数说明)
5. [依赖关系](#依赖关系)
6. [项目运行方式](#项目运行方式)
7. [技能索引](#技能索引)

---

## 项目概述

**Android Skills** 是一个专门为AI/LLM优化的模块化指令和资源仓库，旨在帮助大语言模型更好地理解和执行遵循Android开发最佳实践的特定模式。该项目遵循 [open-standard agent skills](https://agentskills.io/home) 规范，使用Markdown文件（SKILL.md）提供任务的技术规范，为LLM提供专业领域和工作流程的基础信息。

### 核心目标

- 为LLM表现不佳的用例和工作流程提供优化指导
- 提供来自 developer.android.com 的权威Android开发最佳实践
- 以模块化、可组合的方式组织技能指令

### 项目特点

| 特性 | 描述 |
|------|------|
| **模块化设计** | 每个技能独立封装，可单独安装使用 |
| **AI优化** | 专为LLM理解和使用而设计的文档结构 |
| **权威来源** | 基于官方Android开发者文档 |
| **开放标准** | 遵循 agent skills 开放规范 |

---

## 整体架构

### 目录结构

```
/workspace/
├── .github/                    # GitHub配置
│   ├── ISSUE_TEMPLATE/         # Issue模板
│   └── workflows/              # CI/CD工作流
├── build/                      # 构建相关技能
│   └── agp/
│       └── agp-9-upgrade/      # AGP 9升级技能
├── camera/                     # 相机相关技能
│   └── camera1-to-camerax/     # Camera1迁移到CameraX
├── device-ai/                  # 设备AI技能
│   └── appfunctions/           # AppFunctions集成
├── devtools/                   # 开发工具
│   └── android-cli/            # Android CLI工具
├── identity/                   # 身份认证
│   └── verified-email/         # 验证邮箱功能
├── jetpack-compose/            # Jetpack Compose相关
│   ├── adaptive/               # 自适应UI
│   ├── migration/              # 迁移指南
│   └── theming/                # 主题样式
├── navigation/                 # 导航相关
│   └── navigation-3/           # Navigation 3
├── performance/                # 性能优化
│   └── r8-analyzer/            # R8分析器
├── play/                       # Google Play相关
│   ├── engage-sdk-integration/ # Engage SDK集成
│   └── play-billing-library-version-upgrade/  # Play Billing升级
├── profilers/                  # 性能分析工具
│   ├── perfetto-sql/           # Perfetto SQL
│   └── perfetto-trace-analysis/# Perfetto跟踪分析
├── system/                     # 系统相关
│   └── edge-to-edge/           # 边到边显示
├── testing/                    # 测试相关
│   └── testing-setup/          # 测试设置
├── wear/                       # Wear OS相关
│   └── jetpack-compose-m3/     # Wear Compose Material3
├── xr/                         # XR相关
│   └── display-glasses-with-jetpack-compose-glimmer/  # XR显示眼镜
├── LICENSE.txt                 # 许可证文件
└── README.md                   # 项目说明
```

### 架构层次图

```
┌─────────────────────────────────────────────────────────────────┐
│                     Android Skills Repository                     │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  Build &    │  │   UI &      │  │  Platform   │              │
│  │  Config     │  │  Compose    │  │  Services   │              │
│  │  ─────────  │  │  ─────────  │  │  ─────────  │              │
│  │  • AGP      │  │  • Adaptive │  │  • Play     │              │
│  │  • R8       │  │  • Theming  │  │  • Billing  │              │
│  │             │  │  • Nav 3    │  │  • Engage   │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  Device &   │  │  Identity   │  │   DevOps    │              │
│  │  Hardware   │  │  & Auth     │  │  & Testing  │              │
│  │  ─────────  │  │  ─────────  │  │  ─────────  │              │
│  │  • Camera   │  │  • Verified │  │  • Testing  │              │
│  │  • Wear OS  │  │    Email    │  │  • Perfetto │              │
│  │  • XR       │  │  • Passkeys │  │  • Android  │              │
│  │             │  │             │  │    CLI      │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
└─────────────────────────────────────────────────────────────────┘
```

---

## 主要模块职责

### 1. 构建与配置模块 (build/)

#### agp-9-upgrade
**职责**: 升级或迁移Android项目使用Android Gradle Plugin (AGP) 版本9

**核心功能**:
- AGP版本检测与升级指导
- 依赖更新（KSP、Hilt等）
- 迁移到内置Kotlin
- 新AGP DSL迁移
- kapt到KSP迁移
- BuildConfig处理

**适用场景**: 不适用于Kotlin Multiplatform (KMP) 项目

---

### 2. 相机模块 (camera/)

#### camera1-to-camerax
**职责**: 将遗留Android相机实现（Camera1或原始Camera2 API）迁移到CameraX

**核心功能**:
- CameraX依赖添加
- ProcessCameraProvider初始化
- Preview与Tap-to-Focus实现
- 照片捕获
- 相机切换

**关键约束**:
- 不手动管理相机生命周期
- 不手动计算焦点矩阵
- 必须关闭ImageProxy

---

### 3. 设备AI模块 (device-ai/)

#### appfunctions
**职责**: 分析Android应用以识别关键用户工作流，生成Kotlin代码将其暴露给Android系统

**核心功能**:
- 功能发现与分析
- 实现与配置
- KDoc优化
- ADB测试与调试

**前置条件**:
- targetSdk 36或更新
- compileSdk 37或更新

---

### 4. 开发工具模块 (devtools/)

#### android-cli
**职责**: 使用`android`命令行工具编排Android开发任务

**核心功能**:
- 项目创建
- SDK管理
- 设备交互
- 文档搜索
- 模拟器管理
- 截图捕获
- UI布局检查

**命令结构**:
```
android [COMMAND]
├── create     # 创建新项目
├── docs       # 文档搜索
├── emulator   # 模拟器命令
├── run        # 部署应用
├── sdk        # SDK管理
├── skills     # 技能管理
└── studio     # Android Studio命令
```

---

### 5. 身份认证模块 (identity/)

#### verified-email
**职责**: 提供在Android Credential Manager API上实现验证邮箱检索的完整工作流

**核心功能**:
- Digital Credential请求构建
- OpenID4VP请求格式化
- 响应解析
- 服务端验证指导
- Passkey创建

**安全要求**:
- 必须进行服务端验证
- 每次请求生成唯一nonce
- 验证issuer和签名

---

### 6. Jetpack Compose模块 (jetpack-compose/)

#### adaptive
**职责**: 使应用UI适应不同Android设备（手机、平板、可折叠设备、桌面、TV、Auto、XR）

**核心功能**:
- 自适应导航栏
- 多窗格布局（Navigation3 Scenes）
- 自适应垂直列表
- 滚动时隐藏App Bars

**前置条件**:
- 使用Compose
- 使用Navigation 3

#### migrate-xml-views-to-jetpack-compose
**职责**: 提供将Android XML View迁移到Jetpack Compose的结构化工作流

**迁移步骤**:
1. 识别最佳XML候选
2. 分析项目和布局
3. 创建计划
4. 捕获XML View UI
5. 设置Compose依赖
6. 设置Compose主题
7. 迁移XML布局
8. 验证迁移
9. 替换使用
10. 删除XML代码

#### styles
**职责**: 将Jetpack Compose Styles API集成到Android项目

**核心功能**:
- 组件主题设置
- 自定义组件样式化
- Modifier.styleable使用
- 状态和过渡配置

**前置条件**:
- compileSdk 37+
- foundation 1.12.0-alpha01+

---

### 7. 导航模块 (navigation/)

#### navigation-3
**职责**: 学习如何安装和迁移到Jetpack Navigation 3

**核心功能**:
- Navigation 2到3迁移
- 深链接处理
- 多backstack支持
- Scenes（对话框、底部抽屉、列表详情、双窗格）
- 条件导航
- 结果返回
- Hilt/ViewModel集成

**关键组件**:
- `NavKey` - 导航键
- `NavDisplay` - 导航显示
- `SceneStrategy` - 场景策略

---

### 8. 性能模块 (performance/)

#### r8-analyzer
**职责**: 分析Android构建文件和R8 keep规则以识别冗余

**分析路径**:
- **Path A (定量)**: R8 >= 9.3.7-dev，使用配置分析器
- **Path B (启发式)**: R8 < 9.3.7-dev，手动评估

**核心功能**:
- 冗余规则识别
- 过宽包级规则检测
- 库消费者keep规则覆盖检测

---

### 9. Google Play模块 (play/)

#### engage-sdk-integration
**职责**: 帮助开发者集成、调试和解决Play Engage SDK实现问题

**工作流程**:
1. 识别垂直领域和集群
2. 生成结构化样板代码
3. 建议实体映射
4. 建议数据源
5. Gradle和Manifest更新
6. 调试

**支持的垂直领域**:
- Food, Watch, Listen, Read
- Shopping, Social, Travel
- Health & Fitness, TV, Other

#### play-billing-library-version-upgrade
**职责**: 升级Google Play Billing Library到最新稳定版本

**迁移阶段**:
1. 发现与态势感知
2. 上下文文档映射与规划
3. 执行指令
4. 最终验证

---

### 10. 性能分析器模块 (profilers/)

#### perfetto-sql
**职责**: 将自然语言数据意图转换为语法有效的Perfetto SQL查询

**核心原则**:
- 幂等性保证
- SPAN_JOIN安全使用
- 唯一标识符使用（utid/upid）
- GLOB代替LIKE

#### perfetto-trace-analysis
**职责**: 分析Perfetto跟踪以查找延迟、内存或卡顿问题的根本原因

**调查协议**:
1. 制定假设
2. 计划和收集数据
3. 分析和深入
4. 穷尽调查

---

### 11. 系统模块 (system/)

#### edge-to-edge
**职责**: 迁移Jetpack Compose应用以添加自适应边到边支持

**核心功能**:
- 系统insets应用
- IME处理
- 导航栏对比度
- 列表处理
- 对话框处理

**前置条件**:
- 使用Jetpack Compose
- targetSdk 35+

---

### 12. 测试模块 (testing/)

#### testing-setup
**职责**: 分析并为原生Android应用创建测试策略

**测试类型**:
- 单元测试
- UI测试
- 截图测试
- 端到端测试

**框架支持**:
- JUnit4/5
- Espresso
- Compose Testing APIs
- Robolectric
- UI Automator
- Hilt/Koin

---

### 13. Wear OS模块 (wear/)

#### jetpack-compose-m3
**职责**: 使用Wear OS Compose Material3的专家指导

**核心组件**:
- `AppScaffold` / `ScreenScaffold`
- `TransformingLazyColumn`
- `EdgeButton`
- `TitleChip`

**前置条件**:
- Wear Compose Material3最新稳定版
- Kotlin 2.0.0+
- minSdk 25+

---

### 14. XR模块 (xr/)

#### display-glasses-with-jetpack-compose-glimmer
**职责**: 使用Jetpack Compose Glimmer UI工具包开发投影Android XR应用

**核心组件**:
- `GlimmerTheme`
- `Card`, `Button`, `TitleChip`
- `List`, `Stack`
- `Icon`, `Text`

**设计原则**:
- 使用纯黑背景
- 底部对齐UI
- 一次显示一个主要信息
- 70%色调对比度

---

## 关键类与函数说明

### Skill元数据结构

每个SKILL.md文件都包含以下YAML前置元数据：

```yaml
---
name: skill-name                    # 技能唯一标识
description: Skill description      # 技能描述
license: Complete terms in LICENSE.txt
metadata:
  author: Google LLC
  last-updated: 'YYYY-MM-DD'
  keywords:
    - keyword1
    - keyword2
---
```

### 核心函数模式

#### 1. ProcessCameraProvider初始化模式 (CameraX)

```kotlin
val cameraProviderFuture = ProcessCameraProvider.getInstance(context)
cameraProviderFuture.addListener({
    val cameraProvider = cameraProviderFuture.get()
    val cameraSelector = CameraSelector.Builder()
        .requireLensFacing(CameraSelector.LENS_FACING_BACK)
        .build()
    cameraProvider.bindToLifecycle(lifecycleOwner, cameraSelector, preview, imageCapture)
}, ContextCompat.getMainExecutor(context))
```

#### 2. Digital Credential请求模式 (Verified Email)

```kotlin
val getDigitalCredentialOption = GetDigitalCredentialOption(requestJson = openId4vpRequest)
val request = GetCredentialRequest(listOf(getDigitalCredentialOption))
val result = credentialManager.getCredential(activity, request)
```

#### 3. NavigationSuiteScaffold模式 (Adaptive)

```kotlin
NavigationSuiteScaffold(
    navigationSuiteItems = navItems,
    state = scaffoldVisibilityState
) {
    // Main content
}
```

#### 4. TransformingLazyColumn模式 (Wear OS)

```kotlin
TransformingLazyColumn(
    state = columnState,
    contentPadding = contentPadding
) {
    item {
        Button(
            modifier = Modifier
                .fillMaxWidth()
                .transformedHeight(this, transformationSpec)
        ) { /* ... */ }
    }
}
```

---

## 依赖关系

### 模块依赖图

```
┌─────────────────────────────────────────────────────────────┐
│                      External Dependencies                   │
├─────────────────────────────────────────────────────────────┤
│  Android SDK  │  Kotlin  │  Gradle  │  Compose  │  Play SDK │
└───────┬───────┴────┬─────┴────┬─────┴─────┬─────┴─────┬─────┘
        │            │           │           │           │
        ▼            ▼           ▼           ▼           ▼
┌─────────────────────────────────────────────────────────────┐
│                    Android Skills Modules                     │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐                │
│  │  Build  │────▶│ Compose │────▶│   Nav   │                │
│  └─────────┘     └─────────┘     └─────────┘                │
│       │              │               │                       │
│       ▼              ▼               ▼                       │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐                │
│  │   R8    │     │ Adaptive│     │ Testing │                │
│  └─────────┘     └─────────┘     └─────────┘                │
│                      │                                       │
│                      ▼                                       │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐                │
│  │  Wear   │◀────│  Theming│────▶│   XR    │                │
│  └─────────┘     └─────────┘     └─────────┘                │
│                                                               │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐                │
│  │ Camera  │     │ Identity│     │  Play   │                │
│  └─────────┘     └─────────┘     └─────────┘                │
│                                                               │
│  ┌─────────┐     ┌─────────┐                                 │
│  │Perfetto │     │  CLI    │                                 │
│  └─────────┘     └─────────┘                                 │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### 关键依赖版本要求

| 技能模块 | 依赖要求 |
|----------|----------|
| agp-9-upgrade | AGP 9.0+, KSP 2.3.6+, Hilt 2.59.2+ |
| camera1-to-camerax | CameraX 1.3.0+ (互操作), 1.5.0+ (Compose) |
| appfunctions | targetSdk 36+, compileSdk 37+ |
| verified-email | SDK 28+, GMS 25.49.x+ |
| styles | compileSdk 37+, foundation 1.12.0-alpha01+ |
| edge-to-edge | targetSdk 35+ |
| jetpack-compose-m3 | Kotlin 2.0.0+, minSdk 25+ |
| display-glasses | compileSdk 37+ |

---

## 项目运行方式

### 安装Android CLI

**Linux**:
```bash
curl -fsSL https://dl.google.com/android/cli/latest/linux_x86_64/install.sh | bash
```

**Mac ARM**:
```bash
curl -fsSL https://dl.google.com/android/cli/latest/darwin_arm64/install.sh | bash
```

**Mac Intel**:
```bash
curl -fsSL https://dl.google.com/android/cli/latest/darwin_x86_64/install.sh | bash
```

**Windows**:
```cmd
curl -fsSL https://dl.google.com/android/cli/latest/windows_x86_64/install.cmd -o "%TEMP%\i.cmd" && "%TEMP%\i.cmd"
```

### 安装特定技能

```bash
android skills add --skill=r8-analyzer --project=.
```

### 安装所有技能

```bash
android skills add --all
```

### 常用命令

| 命令 | 描述 |
|------|------|
| `android create empty-activity --name="My App"` | 创建新项目 |
| `android sdk install platforms/android-34` | 安装SDK |
| `android emulator create` | 创建虚拟设备 |
| `android run --apks=app.apk` | 运行应用 |
| `android docs search <query>` | 搜索文档 |
| `android layout` | 检查UI布局 |
| `android screenshot` | 截图 |

---

## 技能索引

### 按类别分类

#### 构建与配置
| 技能名称 | 描述 | 关键词 |
|----------|------|--------|
| agp-9-upgrade | AGP 9升级迁移 | AGP 9, Migration, DSL |
| r8-analyzer | R8规则分析 | R8, proguard, keep rules |

#### UI与Compose
| 技能名称 | 描述 | 关键词 |
|----------|------|--------|
| adaptive | 自适应UI | Grid, FlexBox, MediaQuery |
| migrate-xml-views-to-jetpack-compose | XML到Compose迁移 | migration, XML, Views |
| styles | Compose样式API | Styles, Theming, Modifier.styleable |
| edge-to-edge | 边到边显示 | system bars, insets |

#### 导航
| 技能名称 | 描述 | 关键词 |
|----------|------|--------|
| navigation-3 | Navigation 3 | NavKey, NavDisplay, Scenes |

#### 平台服务
| 技能名称 | 描述 | 关键词 |
|----------|------|--------|
| play-billing-library-version-upgrade | Play Billing升级 | PBL, upgrade, migration |
| engage-sdk-integration | Engage SDK集成 | engage, play engage |

#### 设备与硬件
| 技能名称 | 描述 | 关键词 |
|----------|------|--------|
| camera1-to-camerax | 相机迁移 | CameraX, Camera1, Lifecycle |
| jetpack-compose-m3 | Wear OS Material3 | Wear OS, Compose, Material3 |
| display-glasses-with-jetpack-compose-glimmer | XR显示眼镜 | XR, Glimmer, Projected Activity |

#### 身份认证
| 技能名称 | 描述 | 关键词 |
|----------|------|--------|
| verified-email | 验证邮箱 | Credential Manager, Digital Credentials, OTP-less |

#### AI与智能功能
| 技能名称 | 描述 | 关键词 |
|----------|------|--------|
| appfunctions | AppFunctions集成 | AppFunctions, KSP, AI, MCP |

#### 开发工具
| 技能名称 | 描述 | 关键词 |
|----------|------|--------|
| android-cli | Android CLI工具 | sdk, emulator, skills |
| testing-setup | 测试设置 | testing, ui tests, screenshot |
| perfetto-sql | Perfetto SQL | SQL, Query, SPAN_JOIN |
| perfetto-trace-analysis | Perfetto跟踪分析 | profiling, jank, bottleneck |

---

## 附录

### A. 许可证

本项目采用 Apache License 2.0 许可证。详见 [LICENSE.txt](LICENSE.txt)。

### B. 贡献指南

- 通过GitHub Issue提供反馈、报告问题或提出新技能请求
- 目前不接受公开贡献

### C. 社区准则

本项目遵循 [Google's Open Source Community Guidelines](https://opensource.google/conduct/)。

### D. 相关链接

- [Android Skills 官方文档](https://developer.android.com/tools/agents/android-skills)
- [Android CLI 文档](https://developer.android.com/tools/agents/android-cli)
- [Android Studio Skills](https://developer.android.com/studio/gemini/skills)
- [Agent Skills 开放标准](https://agentskills.io/home)

---

*本文档由 Code Wiki Generator 自动生成*
