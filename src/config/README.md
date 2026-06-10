# 配置文件说明

本目录包含 Firefly 主题的所有配置文件，采用模块化设计，每个文件负责特定的功能模块。

## 📁 配置文件结构

```
src/config/
├── index.ts                  # 配置索引文件 - 统一导出
├── siteConfig.ts             # 站点基础配置
├── analyticsConfig.ts        # 统计分析配置（Google Analytics、Umami、51la 等）
├── announcementConfig.ts     # 公告配置
├── backgroundWallpaper.ts    # 背景壁纸配置
├── commentConfig.ts          # 评论系统配置
├── coverImageConfig.ts       # 封面图配置
├── effectsConfig.ts          # 动画特效配置（樱花等）
├── expressiveCodeConfig.ts   # 代码高亮配置
├── fontConfig.ts             # 字体配置
├── footerConfig.ts           # 页脚配置
├── friendsConfig.ts          # 友链配置
├── galleryConfig.ts          # 相册配置
├── licenseConfig.ts          # 许可证配置
├── musicConfig.ts            # 音乐播放器配置
├── navBarConfig.ts           # 导航栏配置（含 LinkPresets 链接预设）
├── pioConfig.ts              # 看板娘配置（Spine、Live2D）
├── plantumlConfig.ts         # PlantUML 图表配置
├── profileConfig.ts          # 用户资料配置
├── sidebarConfig.ts          # 侧边栏布局配置
├── sponsorConfig.ts          # 赞助配置
└── README.md                 # 本文件
```

## 🚀 使用方式

### 推荐：使用配置索引（统一导入）
```typescript
import { siteConfig, profileConfig } from "@/config";
```

### 直接导入单个配置
```typescript
import { siteConfig } from "@/config/siteConfig";
import { profileConfig } from "@/config/profileConfig";
```

## 📋 配置文件列表

├── index.ts                  # 配置索引文件 - 统一导出
├── siteConfig.ts             # 站点基础配置
├── analyticsConfig.ts        # 统计分析配置（Google Analytics、Umami、51la 等）
├── announcementConfig.ts     # 公告配置
├── backgroundWallpaper.ts    # 背景壁纸配置
├── commentConfig.ts          # 评论系统配置
├── coverImageConfig.ts       # 封面图配置
├── effectsConfig.ts          # 动画特效配置（樱花等）
├── expressiveCodeConfig.ts   # 代码高亮配置
├── fontConfig.ts             # 字体配置
├── footerConfig.ts           # 页脚配置
├── friendsConfig.ts          # 友链配置
├── galleryConfig.ts          # 相册配置
├── licenseConfig.ts          # 许可证配置
├── musicConfig.ts            # 音乐播放器配置
├── navBarConfig.ts           # 导航栏配置（含 LinkPresets 链接预设）
├── pioConfig.ts              # 看板娘配置（Spine、Live2D）
├── plantumlConfig.ts         # PlantUML 图表配置
├── profileConfig.ts          # 用户资料配置
├── sidebarConfig.ts          # 侧边栏布局配置
├── sponsorConfig.ts          # 赞助配置
└── README.md                 # 本文件

## 📝 说明

- 所有配置文件均可通过 `index.ts` 统一导入
- 每个配置文件对应 `types/` 目录下的独立类型定义文件
- `siteConfig.ts` 只保留站点核心信息，不聚合其他模块配置
- `navBarConfig.ts` 底部的 `LinkPresets` 可自由自定义导航栏链接的名称、图标和 URL
