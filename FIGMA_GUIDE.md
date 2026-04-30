# Figma MCP 集成指南

本文档定义了如何将 Figma 设计转换为本项目代码的规则，使用 Figma Model Context Protocol (MCP) 服务器。

## 设计系统结构

### 1. 设计令牌 (Design Tokens)

**颜色定义位置**: `src/theme/colors.ts`

```typescript
// 颜色令牌结构
export const Colors = {
  // 主色调
  primary: '#A897E3',          // 主要颜色（紫色）

  // 中性色
  white: '#FFFFFF',
  black: '#000000',

  // 文本颜色
  text: {
    primary: '#2E2A36',        // 主要文本
    tertiary: '#99959E',       // 第三级文本
    quaternary: '#66616D',     // 第四级文本
    inverse: '#FFFFFF',        // 反色文本
  },

  // 背景颜色
  background: {
    primary: '#FFFFFF',
    secondary: '#FAF9FB',
    tertiary: '#2E2A36',
  },

  // 边框颜色
  border: {
    light: '#F3F4F6',
    default: '#99959E',
    dark: '#D1D5DB',
  },

  // 状态颜色
  success: '#10B981',
  warning: '#F59E0B',
  error: '#EF4444',
  info: '#3B82F6',

  // 健康状态颜色
  health: {
    excellent: '#A897E3',
    mild: '#FFB84D',
    moderate: '#FF9F4D',
    severe: '#FF6B6B',
  },

  // 其他功能性颜色
  disabled: '#D1D5DB',
  placeholder: '#D1D5DB',
  shadow: '#000000',
  gauge: {
    dark: '#6B5B95',           // 仪表盘深色
    light: '#F0F0F0',          // 仪表盘浅色
    track: '#F6F4FA',          // 滑块轨道色
  },

  // 检测详情页面颜色
  detection: {
    abnormal: '#FF6B6B',       // 异常状态
    lightBackground: '#FFF0F0', // 异常提示背景
  },
};
```

**间距和尺寸缩放**: `src/utils/scale.ts`

- 设计稿基准尺寸: **393 x 852** (iPhone 14 Pro)
- 使用 `px()` 进行水平尺寸缩放
- 使用 `pxH()` 进行垂直尺寸缩放
- 使用 `scaleFontSize()` 进行字体缩放

### 2. 框架和库

| 类别 | 技术栈 |
|------|--------|
| UI 框架 | React Native 0.80.2 |
| 语言 | TypeScript 5.0.4 |
| React 版本 | React 19.1.0 |
| 状态管理 | Zustand |
| 导航 | React Navigation v7 |
| HTTP 请求 | Axios (封装在 `@/api/http.ts`) |
| 图片优化 | react-native-fast-image |
| SVG 支持 | react-native-svg + react-native-svg-transformer |

### 3. 资产管理

**资产存储位置**:
```
src/assets/
├── app/              # 通用应用图标和图片
├── fasheng/          # 品牌相关资产
├── device/           # 设备相关图标 (SVG)
├── hairHealth/       # 毛发健康模块图标 (SVG)
├── hairLoss/         # 脱发检测模块图标 (SVG)
├── hairdiary/        # 毛发日记模块图标 (SVG)
├── message/          # 消息模块图标 (SVG)
└── activity/         # 活动相关图标 (SVG)
```

**资产使用规范**:
```typescript
// PNG 图片引入
const icon = require('@/assets/app/icon.png');

// SVG 图标引入 (作为组件)
import ArrowIcon from '@/assets/hairdiary/arrow-right.svg';
```

### 4. 图标系统

- **格式**: 优先使用 SVG，复杂图片使用 PNG
- **存储**: 按功能模块分类存储在 `src/assets/[module]/` 目录
- **命名规范**: kebab-case (如 `arrow-right.svg`, `camera-icon.svg`)
- **SVG 使用**: 通过 `react-native-svg-transformer` 作为 React 组件导入

### 5. 样式方法

**样式工具**: `src/utils/StyleSheet.ts`

```typescript
import StyleSheet from '@/utils/StyleSheet';
import { Colors } from '@/theme';

const styles = StyleSheet.create({
  container: {
    width: 343,        // 自动转换为 px(343)
    height: 200,       // 自动转换为 pxH(200)
    padding: 16,       // 自动转换为 px(16)
    fontSize: 14,      // 自动转换为 scaleFontSize(14)
    borderRadius: 12,  // 保持固定（小于48的值不缩放）
    backgroundColor: Colors.background.primary,
    color: Colors.text.primary,
  }
});
```

**响应式设计**:
- 所有尺寸值在 StyleSheet.create 中自动按设计稿比例缩放
- 小于等于 48 的数值保持固定（如圆角、边框、小图标）
- 大于 48 的数值按屏幕比例缩放

## Figma MCP 集成流程

### 必须遵循的流程（不可跳过）

1. **首先运行 `get_design_context`** 获取指定节点的结构化表示
2. **如果响应过大或被截断**，运行 `get_metadata` 获取高层节点映射，然后仅重新获取所需节点
3. **运行 `get_screenshot`** 获取正在实现的节点变体的视觉参考
4. **只有在获得 `get_design_context` 和 `get_screenshot` 之后**，才下载所需资产并开始实现
5. **转换输出**（通常是 React + Tailwind）为本项目的约定、样式和框架
6. **完成前验证** - 对照 Figma 检查 1:1 的外观和行为

### 实现规则

**转换 Figma MCP 输出**:
- IMPORTANT: Figma MCP 输出（React + Tailwind）作为设计和行为的表示，**不是**最终代码风格
- 将 Tailwind 工具类替换为项目的 `StyleSheet.create()` 样式
- 从 `src/components/` 复用现有组件，而不是重复功能
- 始终使用项目的颜色系统、排版比例和间距令牌
- 遵循现有的路由、状态管理和数据获取模式

**颜色映射**:
```typescript
// Figma 颜色 → 项目颜色
// 主色/紫色 → Colors.primary
// 文本色 → Colors.text.primary / tertiary / quaternary
// 背景色 → Colors.background.primary / secondary
// 边框色 → Colors.border.light / default
```

**组件结构**:
```typescript
import React from 'react';
import { View, Text, TouchableOpacity } from 'react-native';
import StyleSheet from '@/utils/StyleSheet';
import { Colors } from '@/theme';

interface ComponentNameProps {
  // 定义所有 props 类型
}

const ComponentName: React.FC<ComponentNameProps> = ({ /* props */ }) => {
  return (
    <View style={styles.container}>
      {/* 组件内容 */}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    // 样式定义
  },
});

export default ComponentName;
```

### 资产处理规则

- IMPORTANT: 如果 Figma MCP 服务器返回图片或 SVG 的 localhost 源，**直接使用该源**
- IMPORTANT: **不要导入/添加新的图标包** - 所有资产应该在 Figma 负载中
- IMPORTANT: 如果提供了 localhost 源，**不要使用或创建占位符**
- 将下载的资产存储在 `src/assets/[module]/` 目录

### 项目特定约定

**文件组织**:
```
src/pages/NewPage/
├── index.tsx           # 主页面组件
└── components/         # 页面专属组件
    ├── ComponentA.tsx
    └── ComponentB.tsx
```

**导入顺序**:
```typescript
// 1. React/React Native
import React, { useState, useCallback } from 'react';
import { View, Text, FlatList } from 'react-native';

// 2. 第三方库
import { useNavigation } from '@react-navigation/native';

// 3. 项目内部模块
import StyleSheet from '@/utils/StyleSheet';
import { Colors } from '@/theme';
import { Header } from '@/components';
import { useUserStore } from '@/stores/userStore';
```

**导航类型**:
- 所有新路由必须在 `src/types/navigation.ts` 中定义类型
- 使用 `NativeStackScreenProps<RootStackParamList, 'ScreenName'>` 定义页面 Props

**API 集成**:
- 在 `src/api/` 目录创建 API 函数
- 使用 `request` 实例（from `@/api/http.ts`）
- API 响应格式: `{ code: number, msg: string, data: any }`

### 质量验证清单

实现 Figma 设计时，确保:
- [ ] 使用 `Colors` 对象而非硬编码颜色值
- [ ] 使用 `StyleSheet.create()` 而非内联样式
- [ ] 复用 `src/components/` 中的现有组件
- [ ] 为所有 props 添加 TypeScript 类型定义
- [ ] 使用 `FlatList` 而非 `ScrollView + map` 渲染列表
- [ ] 使用 `useCallback` 包装回调函数
- [ ] 对照 Figma 截图验证 1:1 视觉一致性
- [ ] 在 iOS 和 Android 上测试组件

---

**最后更新**：2025-12-31
**适用版本**：React Native 0.80.2 | React 19.1.0
