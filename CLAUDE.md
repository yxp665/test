# HairScope React Native 项目规则

## 项目概述
React Native 0.80.2 | React 19.1.0 | TypeScript 5.0.4
**日常分支**：`main`（版本发布走 `release/x.x.x`）

移动应用项目，包含社交功能、头发检测等核心功能。

## 快速命令

| 场景 | 命令 |
|------|------|
| 启动 Metro | `yarn start` |
| 运行 iOS | `yarn ios` |
| 运行 Android | `yarn android` |
| 运行 Web 版 | `yarn web` |
| 构建 Android Release | `yarn build:android` |
| 构建 iOS Release | `yarn build:ios` |
| 版本号升级 | `yarn bump` |
| Lint | `yarn lint` |

API 共享契约：见工作区根 [../docs/api-contract.md](../docs/api-contract.md)

---

## 核心规范

### TypeScript
- **必须使用 `.tsx` / `.ts`**，避免 `any`
- 组件 Props 使用 `interface` 定义
- 导航参数在 `src/types/navigation.ts` 定义
- **命名规范**：
  - 组件文件：`PascalCase.tsx`
  - 工具函数：`camelCase.ts`
  - Store：`camelCase.ts` (导出 `useXxxStore`)

### React Native 最佳实践
- 函数组件 + Hooks，回调用 `useCallback`
- 列表**必须**用 `FlatList`/`SectionList`（不用 `ScrollView + map`）
- 样式用 `@/utils/StyleSheet`，颜色从 `@/theme/Colors` 导入
- 图片优先 `react-native-fast-image`
- **弹窗必须用 `CustomModal`**，禁止使用原生 `Modal`（iOS 多弹窗会导致页面卡死）

### 状态管理
- 全局状态：Zustand（`src/stores/`）
- 本地状态：`useState` / `useReducer`

### API 调用
- 统一在 `src/api/` 定义，使用 `request` 实例
- 错误处理：`try-catch` + `showError` + `console.error`
- 参数：GET 用 `params`，POST/PUT 用 `data`

### 导航
- 路由类型在 `src/types/navigation.ts` 定义
- 页面 Props：`NativeStackScreenProps<RootStackParamList, 'ScreenName'>`
- 跳转：`navigation.navigate('ScreenName', { params })`

### 文件组织
```
src/
├── api/              # API 接口
├── components/       # 通用组件
├── pages/PageName/
│   ├── index.tsx
│   └── components/   # 页面专属组件
├── stores/           # Zustand stores
├── types/            # TS 类型
├── utils/            # 工具函数
├── theme/            # 主题配置
└── assets/           # 静态资源
```

**导入顺序**：第三方库 → React/RN → 项目模块（按字母排序）

---

## 开发流程

### 新功能
1. 确认需求 → 2. 复用组件/工具 → 3. 定义类型 → 4. 实现（API → UI → 状态 → 路由）→ 5. 测试

### 优化原则
- **DRY**：重复代码立即抽取
- **精简**：合并相似逻辑，提取常量
- **可读性优先**：过度优化会降低可读性时，选择可读性

### 调试
- 控制台错误 + `adb logcat`（Android）
- 检查 API 响应字段是否匹配代码
- 验证类型定义与实际数据一致

---

## 安全与质量

- **不硬编码** API 密钥/token，用 AsyncStorage 存储
- **不输出**完整用户信息/token 到日志
- 所有异步操作 `try-catch`，API 响应用可选链
- 组件/API 函数添加 JSDoc 注释

---

## 快速参考

### 主题色
**位置**: `src/theme/colors.ts`

```typescript
Colors.primary         // #A897E3 主色（紫色）
Colors.text.primary    // #2E2A36 主要文本
Colors.background.primary  // #FFFFFF
Colors.health.excellent    // 健康状态
Colors.disabled       // #D1D5DB
Colors.gauge.*        // 仪表盘颜色
Colors.detection.*    // 检测页颜色
```

### 常用组件
**位置**: `src/components/`

| 组件 | 用途 |
|------|------|
| `CustomModal` | **通用弹窗（替代原生 Modal）** |
| `Header` | 页面头部 |
| `EmptyState` | 空状态 |
| `ConfirmDialog` | 确认弹窗 |
| `FollowButton` | 关注按钮 |
| `RichTextInput` | 富文本输入 |
| `IjkVideoPlayer` | 视频播放器 |

#### CustomModal 使用方式
```tsx
// 底部弹出
<CustomModal isVisible={visible} onBackdropPress={onClose} position="bottom">
  <View>{/* 内容 */}</View>
</CustomModal>

// 居中显示
<CustomModal isVisible={visible} onBackdropPress={onClose} position="center">
  <View>{/* 内容 */}</View>
</CustomModal>
```

> 更多组件见 `src/components/` 目录

### Stores
**位置**: `src/stores/`

- `userStore` (useUserStore) - 用户信息
- `useDeviceStore` - 设备状态
- `useMessageStore` - 消息

### API 响应格式 / 常见字段
见 [../docs/api-contract.md](../docs/api-contract.md)

### Git 提交
中文描述，简洁明了。示例：
- "新增我的关注功能"
- "修复按钮状态显示错误"

---

## 扩展文档

- **Figma 集成**: [FIGMA_GUIDE.md](FIGMA_GUIDE.md)
- **完整颜色系统**: `src/theme/colors.ts`
- **所有组件**: `src/components/`

---

## 踩坑记录

### iOS 多 Modal 导致页面无法点击
- **现象**：关闭弹窗后整个页面无法响应触摸事件，Modal 不会自动销毁
- **原因**：iOS 上原生 Modal 同时存在（即使 `visible={false}`）会拦截触摸事件
- **解决**：**使用 `CustomModal` 替代原生 `Modal`**
  - `CustomModal` 使用绝对定位 View 实现，完全绕开原生 Modal 的坑
  - 条件渲染确保组件真正销毁，不会残留拦截触摸事件
  - 支持 `position="bottom"`（底部弹出）和 `position="center"`（居中显示）

### iOS Modal 内输入框被键盘遮挡/飞出屏幕
- **现象**：底部弹窗中点击输入框，内容区域飞到屏幕外
- **原因**：`react-native-avoid-softinput` 的 `AvoidSoftInputView` 在底部 Modal 中计算偏移错误
- **解决**：改用 `KeyboardAvoidingView` + `behavior={Platform.OS === 'ios' ? 'padding' : undefined}`

### CustomModal 底部弹窗禁止使用 ScrollView 包裹内容
- **现象**：Android 上键盘弹出后弹窗内容飞到屏幕顶部
- **原因**：`CustomModal` 的 `avoidKeyboard` 已通过 `paddingBottom: keyboardOffset` 处理键盘避让。如果内部再用 `ScrollView`，Android 的 `adjustResize` + ScrollView 自身滚动 + CustomModal 的 paddingBottom 会产生三重补偿，内容被推到屏幕外
- **规则**：
  - `position="bottom"` + `avoidKeyboard` 的弹窗，**禁止**用 `ScrollView` 包裹内容
  - 需要点击空白收键盘：用 `TouchableWithoutFeedback onPress={Keyboard.dismiss}` + `View` 包裹
  - 需要键盘不影响按钮点击：用 `TouchableWithoutFeedback`（不会拦截子 `TouchableOpacity` 的事件），**不要**用 `Pressable onPress={Keyboard.dismiss}`（会抢占子组件的第一次点击）
  - `position="center"` 且无 `avoidKeyboard` 的弹窗可以用 `ScrollView`（不存在双重补偿问题）

### 弹窗内按钮需要点两次才能触发
- **现象**：键盘弹出时点击提交按钮，第一次无反应（收键盘），需要点第二次
- **原因有两个**：
  1. 外层用 `Pressable onPress={Keyboard.dismiss}` 包裹时，Pressable 会拦截所有触摸事件，第一次点击被消费掉去收键盘
  2. `Pressable` 组件作为按钮放在 `ScrollView keyboardShouldPersistTaps="always"` 内时，`keyboardShouldPersistTaps` 对 `Pressable` **不生效**（Pressable 用新版 Pressability API，绕过了 ScrollView 的触摸拦截机制）
- **解决**：
  - 收键盘容器：用 `TouchableWithoutFeedback onPress={Keyboard.dismiss}` 替代 `Pressable`
  - 弹窗/ScrollView 内的按钮：**一律用 `TouchableOpacity`，禁止用 `Pressable`**
  - `keyboardShouldPersistTaps="always"` 只对 `TouchableOpacity` / `TouchableHighlight` 等旧版 Touchable 组件有效

---

**最后更新**: 2026-05
