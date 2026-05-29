---
title: 为 Firefly添加 Umami 访问统计卡片
published: 2026-05-29
description: ''
image: ''
tags: [Firefly,Umami,Astro]
category: 'blog'
draft: false 
sourceLink: "https://blog.tianhw.top/posts/fuwari-umami-stats/"
author: "THW’s Blog"
---
> [!NOTE] 
> 本教程基于[THW’s Blog](https://blog.tianhw.top/posts/fuwari-umami-stats/)大佬教程两次创作。

## 开始
了解访客来源与流量趋势是很有必要的。静态博客中配置简单，功能强大的 Umami 往往是我们的首选。

然而，直接挂载 Umami 的分享外链不够直观且破坏页面一致性。本文将教你如何将 Umami 的统计数据以原生组件的形式集成到 Firefly 主题的侧边栏中，让你的博客实时展示访问数据。

## 准备
首先要有一个 Umami，并获得Umami的分享链接。

## 添加组件
在 **src/components/widget/** 目录下创建 **UmamiStats.astro 文件**，代码如下：
```astro
---
import WidgetLayout from "../common/WidgetLayout.astro";

interface Props {
	class?: string;
	style?: string;
}
const { class: className, style } = Astro.props;
---

<WidgetLayout name="统计" class:list={["umami-stats-container", className, "cursor-pointer transition-opacity active:scale-95"]} {style}>
    <a target="_blank" rel="noopener noreferrer" class="block umami-link">
        <div class="text-center py-2">
            <div class="text-3xl font-bold text-neutral-900 dark:text-neutral-100 umami-total-pageviews">-</div>
            <div class="text-sm text-neutral-500 dark:text-neutral-400">总浏览量</div>
        </div>
        <div class="grid grid-cols-2 divide-x divide-neutral-200 dark:divide-neutral-700 text-center pt-2">
            <div class="px-2">
                <div class="text-xl font-bold text-neutral-900 dark:text-neutral-100 umami-total-visits">-</div>
                <div class="text-sm text-neutral-500 dark:text-neutral-400">访问数</div>
            </div>
            <div class="px-2">
                <div class="text-xl font-bold text-neutral-900 dark:text-neutral-100 umami-total-visitors">-</div>
                <div class="text-sm text-neutral-500 dark:text-neutral-400">游客数</div>
            </div>
        </div>
    </a>
</WidgetLayout>

<script>
const UMAMI_CONFIG = {
    shareUrl: '你的方享链接',
};

let __UMAMI_INTERNAL = {
    baseUrl: '',
    websiteId: '',
    shareToken: '',
    shareId: '',
    isReady: false
};

const FALLBACK_STATS = {
    pageviews: 1000,
    visits: 1000,
    visitors: 1000,
};

async function initUmamiConfig() {
    try {
        const sharePath = UMAMI_CONFIG.shareUrl.split('/share/')[1];
        if (!sharePath) throw new Error('Invalid Umami Share URL');

        // 自动识别是官方云服务还是自建域名
        let apiBase = '';
        if (UMAMI_CONFIG.shareUrl.includes('cloud.umami.is') || UMAMI_CONFIG.shareUrl.includes('analytics.umami.is')) {
            const region = UMAMI_CONFIG.shareUrl.includes('/analytics/eu/') ? 'eu' : 'us';
            apiBase = `https://cloud.umami.is/analytics/${region}/api`;
        } else {
            // 自建域名动态提取基础 API 路径
            const urlObj = new URL(UMAMI_CONFIG.shareUrl);
            apiBase = `${urlObj.origin}/api`;
        }

        const res = await fetch(`${apiBase}/share/${sharePath}`);
        if (!res.ok) throw new Error(`Failed to fetch share config: ${res.status}`);
        const data = await res.json();

        __UMAMI_INTERNAL = {
            baseUrl: apiBase,
            websiteId: data.websiteId,
            shareToken: data.token,
            shareId: data.shareId,
            isReady: true
        };

        // 更新页面上所有统计组件的跳转链接
        const links = document.querySelectorAll('.umami-link');
        links.forEach(link => link.setAttribute('href', UMAMI_CONFIG.shareUrl));

    } catch (e) {
        console.error('Umami Config Init Failed:', e);
    }
}

function formatNumber(num: number): string {
    if (num >= 1000000) {
        return (num / 1000000).toFixed(1) + 'M';
    } else if (num >= 1000) {
        return (num / 1000).toFixed(1) + 'K';
    }
    return Math.round(num).toString();
}

function setStats(values: { pageviews: number; visits: number; visitors: number }) {
    // 抓取页面上所有的统计组件实例
    const pageviewsElements = document.querySelectorAll('.umami-total-pageviews');
    const visitsElements = document.querySelectorAll('.umami-total-visits');
    const visitorsElements = document.querySelectorAll('.umami-total-visitors');

    const easeOutCubic = (t: number) => 1 - Math.pow(1 - t, 3);
    const animHandles = new Map<HTMLElement, number>();

    const animateStat = (el: HTMLElement | null, to: number, duration = 2000) => {
        if (!el) return;

        const prev = animHandles.get(el);
        if (prev) cancelAnimationFrame(prev);

        const from = 0;
        const startTime = performance.now();

        const tick = (now: number) => {
            const elapsed = now - startTime;
            const progress = Math.min(1, elapsed / duration);
            const easedProgress = easeOutCubic(progress);

            const current = from + (to - from) * easedProgress;
            el.textContent = formatNumber(current);

            if (progress < 1) {
                animHandles.set(el, requestAnimationFrame(tick));
            }
        };
        animHandles.set(el, requestAnimationFrame(tick));
    };

    // 群发更新所有节点的数字
    pageviewsElements.forEach(el => animateStat(el as HTMLElement, values.pageviews));
    visitsElements.forEach(el => animateStat(el as HTMLElement, values.visits));
    visitorsElements.forEach(el => animateStat(el as HTMLElement, values.visitors));
}

async function fetchUmamiStats() {
    if (!__UMAMI_INTERNAL.isReady) {
        await initUmamiConfig();
    }

    if (!__UMAMI_INTERNAL.isReady) {
        setStats(FALLBACK_STATS);
        return;
    }

    try {
        const endAt = Date.now();
        const startAt = 0; 
        const url = `${__UMAMI_INTERNAL.baseUrl}/websites/${__UMAMI_INTERNAL.websiteId}/stats?startAt=${startAt}&endAt=${endAt}&unit=hour&timezone=Asia%2FShanghai`;

        const response = await fetch(url, {
            headers: {
                'x-umami-share-context': '1',
                'x-umami-share-token': __UMAMI_INTERNAL.shareToken
            }
        });

        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        const data = await response.json();
        const getValue = (field: any) => (typeof field === 'object' ? field?.value : field) || 0;

        setStats({
            pageviews: getValue(data.pageviews),
            visits: getValue(data.visits),
            visitors: getValue(data.visitors),
        });

    } catch (error) {
        console.error('Umami Fetch Failed:', error);
        setStats(FALLBACK_STATS);
    }
}

let __umamiStatsStarted = false;
function startUmamiStats() {
    if (__umamiStatsStarted) return;
    __umamiStatsStarted = true;
    fetchUmamiStats();
}

function initUmamiStatsVisibility() {
    const containers = document.querySelectorAll('.umami-stats-container');
    const io = new IntersectionObserver((entries) => {
        let isAnyVisible = false;
        entries.forEach(entry => {
            if (entry.isIntersecting) isAnyVisible = true;
        });
        
        if (isAnyVisible) {
            startUmamiStats();
            io.disconnect();
        }
    }, { threshold: 0.1 });

    containers.forEach(container => io.observe(container));
}

initUmamiStatsVisibility();

if (window.swup) {
    window.swup.hooks.on('page:view', () => {
        __umamiStatsStarted = false;
        initUmamiStatsVisibility();
    });
}
</script>
```

## 配置参数
在代码文件的 script 部分，填入你的分享链接
```javascript
<script>
const UMAMI_CONFIG = {
    shareUrl: '你的方享链接',
};
```

## 添加到侧边栏组件
在**src/components/layout/SideBar.astro**中导入并使用此组件：
```
//顶部加入
import UmamiStats from "../widget/UmamiStats.astro";
```
你还需在div前导入,这部分可以查看下方完整代码
```astro
---
import type { MarkdownHeading } from "astro";
import Advertisement from "@/components/widget/Advertisement.astro";
import Announcement from "@/components/widget/Announcement.astro";
import Calendar from "@/components/widget/Calendar.astro";
import Categories from "@/components/widget/Categories.astro";
import Music from "@/components/widget/Music.astro";
import Profile from "@/components/widget/Profile.astro";
import SidebarTOC from "@/components/widget/SidebarTOC.astro";
import SiteStats from "@/components/widget/SiteStats.astro";
import Tags from "@/components/widget/Tags.astro";
import { sidebarLayoutConfig } from "@/config";
import type {
	MobileBottomComponentConfig,
	WidgetComponentConfig,
	WidgetComponentType,
} from "@/types/config";
import UmamiStats from "../widget/UmamiStats.astro";

interface Props {
	class?: string;
	headings?: MarkdownHeading[];
	side?: "left" | "right" | "bottom";
}

// 侧边栏位置常量
const SIDEBAR_SIDE = {
	LEFT: "left",
	RIGHT: "right",
	BOTTOM: "bottom",
} as const;

// 组件位置常量
const COMPONENT_POSITION = {
	TOP: "top",
	STICKY: "sticky",
} as const;

// 动画延迟配置
const ANIMATION_DELAY_UNIT = 50; // ms

// 组件映射表
const componentMap = {
	profile: Profile,
	announcement: Announcement,
	categories: Categories,
	tags: Tags,
	sidebarToc: SidebarTOC,
	advertisement: Advertisement,
	stats: SiteStats,
	calendar: Calendar,
	music: Music,
} satisfies Record<WidgetComponentType, typeof Profile>;

// 获取侧边栏位置
const side = (Astro.props.side ||
	SIDEBAR_SIDE.LEFT) as (typeof SIDEBAR_SIDE)[keyof typeof SIDEBAR_SIDE];
const className = Astro.props.class;

// 根据 side 属性获取对应的组件列表
const getComponents = (): (
	| WidgetComponentConfig
	| MobileBottomComponentConfig
)[] => {
	if (side === SIDEBAR_SIDE.LEFT) {
		return sidebarLayoutConfig.leftComponents;
	}
	if (side === SIDEBAR_SIDE.RIGHT) {
		return sidebarLayoutConfig.rightComponents;
	}
	if (side === SIDEBAR_SIDE.BOTTOM) {
		return sidebarLayoutConfig.mobileBottomComponents;
	}
	return [];
};

// 过滤并排序组件
const filterAndSortComponents = (
	components: (WidgetComponentConfig | MobileBottomComponentConfig)[],
) => {
	return components.filter((comp) => comp.enable);
};

// 分离 top 和 sticky 位置的组件
const getComponentsByPosition = (
	components: (WidgetComponentConfig | MobileBottomComponentConfig)[],
) => {
	const topComponents = components.filter(
		(c) => "position" in c && c.position === COMPONENT_POSITION.TOP,
	) as WidgetComponentConfig[];
	const stickyComponents = components.filter(
		(c) => "position" in c && c.position === COMPONENT_POSITION.STICKY,
	) as WidgetComponentConfig[];
	return { topComponents, stickyComponents };
};

// 获取动画延迟
const getAnimationDelay = (index: number): string => {
	return `${index * ANIMATION_DELAY_UNIT}ms`;
};

// 判断当前页面是否为文章页面
const isPostPage = Astro.url.pathname.includes("/posts/");

// 判断组件在当前页面首屏是否可见
const isComponentInitiallyVisible = (
	config: WidgetComponentConfig | MobileBottomComponentConfig,
): boolean => {
	if (
		"showOnPostPage" in config &&
		config.showOnPostPage === false &&
		isPostPage
	) {
		return false;
	}
	if (
		"showOnNonPostPage" in config &&
		config.showOnNonPostPage === false &&
		!isPostPage
	) {
		return false;
	}
	return true;
};

// 动态构建组件 props
const getComponentProps = (
	config: WidgetComponentConfig | MobileBottomComponentConfig,
	index: number,
): Record<string, unknown> => {
	const baseProps: Record<string, unknown> = {
		class: "onload-animation",
		style: `animation-delay: ${getAnimationDelay(index)}`,
	};

	if ("showOnPostPage" in config && config.showOnPostPage === false) {
		baseProps.class = `${baseProps.class} widget-hide-on-post`;
		if (isPostPage) {
			baseProps.class = `${baseProps.class} hidden`;
		}
	}
	if ("showOnNonPostPage" in config && config.showOnNonPostPage === false) {
		baseProps.class = `${baseProps.class} widget-hide-on-non-post`;
		if (!isPostPage) {
			baseProps.class = `${baseProps.class} hidden`;
		}
	}

	if (config.type === "sidebarToc") {
		return { ...baseProps, headings: Astro.props.headings || [] };
	}

	if (
		config.type === "advertisement" &&
		"configId" in config &&
		config.configId
	) {
		return { ...baseProps, configId: config.configId };
	}

	return baseProps;
};

// 获取所有需要渲染的组件
const allComponents = getComponents();
const filteredComponents = filterAndSortComponents(allComponents);

const isMobileBottom = side === SIDEBAR_SIDE.BOTTOM;
const { topComponents, stickyComponents } = !isMobileBottom
	? getComponentsByPosition(filteredComponents)
	: { topComponents: [], stickyComponents: [] };
const bottomComponents = isMobileBottom ? filteredComponents : [];
const hasInitiallyVisibleTopComponents = topComponents.some(
	isComponentInitiallyVisible,
);

// 为移动端准备分离的组件（为了实现 Profile 在上，统计在下）
let mobileProfileComp: MobileBottomComponentConfig | undefined;
let mobileOtherComponents: MobileBottomComponentConfig[] = [];

if (isMobileBottom) {
	mobileProfileComp = bottomComponents.find((comp) => comp.type === "profile");
	mobileOtherComponents = bottomComponents.filter(
		(comp) => comp.type !== "profile",
	);
}

// 决定是否渲染 sticky 容器（如果有 sticky 组件，或者当前是左侧边栏）
const hasStickyContent =
	stickyComponents.length > 0 || side === SIDEBAR_SIDE.LEFT;
---

{
	(topComponents.length > 0 || stickyComponents.length > 0 || bottomComponents.length > 0 || hasStickyContent) && (
		<div id={`${side}-sidebar`} class:list={[className, "flex flex-col w-full pt-0"]}>
			{/* Mobile bottom components - 移动端布局 */}
			{isMobileBottom ? (
				<div class="flex flex-col w-full gap-4">
					{/* 1. 先渲染个人信息 */}
					{mobileProfileComp && (() => {
						const Component = componentMap[mobileProfileComp.type];
						if (!Component) return null;
						const props = getComponentProps(mobileProfileComp, 0) as any;
						return <Component {...props} />;
					})()}

					{/* 2. 接着渲染统计组件（实现上下互换） */}
					<UmamiStats class="onload-animation" style="animation-delay: 50ms" />
					
					{/* 3. 最后渲染其他的常规组件 */}
					{mobileOtherComponents.map((comp, index) => {
						const Component = componentMap[comp.type];
						if (!Component) return null;
						const props = getComponentProps(comp, index + 2) as any; 
						return <Component {...props} />;
					})}
				</div>
			) : (
				<>
					{/* Top components */}
					{topComponents.length > 0 && (
						<div
							class:list={[
								"flex flex-col w-full gap-4",
								hasInitiallyVisibleTopComponents && "mb-4",
							]}
						>
							{topComponents.map((comp, index) => {
								const Component = componentMap[comp.type];
								if (!Component) return null;
								const props = getComponentProps(comp, index) as any;
								return <Component {...props} />;
							})}
						</div>
					)}

					{/* Sticky components */}
					{hasStickyContent && !isMobileBottom && (
						<div
							id={`${side}-sidebar-sticky`}
							class:list={[
								"flex flex-col w-full mt-0 gap-4",
								"sticky",
								hasInitiallyVisibleTopComponents ? "top-4" : "top-0",
							]}
						>
							{/* PC 端仅在左侧边栏渲染统计，避免两侧重复 */}
							{side === SIDEBAR_SIDE.LEFT && (
								<UmamiStats class="onload-animation" style="animation-delay: 200ms" />
							)}

							{stickyComponents.map((comp, index) => {
								const Component = componentMap[comp.type];
								if (!Component) return null;
								const props = getComponentProps(
									comp,
									topComponents.length + index
								) as any;
								return <Component {...props} />;
							})}
						</div>
					)}
				</>
			)}
		</div>
	)
}
```
## 结尾
通过以上步骤，你就成功为 Firefly 添加了具有丰富交互感、自动解析配置且支持点击查看详情的 Umami 统计卡片。