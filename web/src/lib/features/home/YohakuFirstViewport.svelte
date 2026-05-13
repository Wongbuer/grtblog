<script lang="ts">
	import { resolveHref } from '$lib/shared/utils/resolve-path';
	import { FadeIn } from '$lib/ui/animation';
	import DynamicLucideIcon from '$lib/ui/icons/DynamicLucideIcon.svelte';
	import { ArrowDown, UserRound } from 'lucide-svelte';
	import type { NavMenuItem } from '$lib/features/navigation/types';
	import type {
		HomeFirstViewportThemeConfig,
		HomeHeroSocialLink,
		HomeHeroTemplateNode,
		HomeHeroThemeConfig
	} from './types';

	let {
		config,
		firstViewport,
		navMenus = [],
		siteName = 'Blog',
		fallbackImage = ''
	}: {
		config?: HomeHeroThemeConfig;
		firstViewport?: HomeFirstViewportThemeConfig;
		navMenus?: NavMenuItem[];
		siteName?: string;
		fallbackImage?: string;
	} = $props();

	const defaultTitleTemplate: HomeHeroTemplateNode[] = [
		{ type: 'h1', text: 'Hi! 👋', variant: 'hero_h1_highlight' },
		{ type: 'br' },
		{ type: 'h1', text: "I'm grtsinry43", variant: 'hero_h1_primary' }
	];
	const defaultDescription =
		'Java & JavaScript full-stack developer committed to crafting excellent software.';
	const defaultMottoLines = [
		'热衷于在逻辑与感性的缝隙中构建数字花园。',
		'也许，代码是现代的诗歌，而文字是思想的快照。'
	];
	const defaultSocials: HomeHeroSocialLink[] = [
		{ icon: 'github', name: 'GitHub', href: 'https://github.com/grtinry43' },
		{ icon: 'mail', name: 'Email', href: 'mailto:grtsinry43@outlook.com' },
		{ icon: 'rss', name: 'RSS', href: '/feed' }
	];

	const isExternalHttpHref = (href: string): boolean => /^(https?:|\/\/)/i.test(href);
	const normalizeHref = (href: string): string => (href.startsWith('/') ? resolveHref(href) : href);
	const shouldOpenInNewTab = (href: string): boolean => isExternalHttpHref(href);
	const isActiveNavItem = (href: string): boolean => href === '/' || href === '';

	const extractTitleText = (nodes: HomeHeroTemplateNode[] | undefined): string =>
		(nodes ?? [])
			.map((node) => (node.type === 'br' ? ' ' : (node.text ?? '')))
			.join(' ')
			.replace(/\s+/g, ' ')
			.trim();

	const titleTemplate = $derived(
		config?.titleTemplate && config.titleTemplate.length > 0
			? config.titleTemplate
			: defaultTitleTemplate
	);
	const titleFromTemplate = $derived(extractTitleText(titleTemplate));
	const titleText = $derived(titleFromTemplate || siteName || 'Blog');
	const description = $derived(config?.description || defaultDescription);
	const avatarUrl = $derived(config?.avatarUrl || fallbackImage);
	const siteInitial = $derived(titleText.trim().slice(0, 1).toUpperCase() || 'G');
	const mottoLines = $derived(
		config?.mottoLines && config.mottoLines.length > 0 ? config.mottoLines : defaultMottoLines
	);
	const socials = $derived(
		config?.socials && config.socials.length > 0 ? config.socials : defaultSocials
	);
	const showTopNav = $derived(firstViewport?.showTopNav ?? true);
	const showScrollHint = $derived(firstViewport?.showScrollHint ?? true);
	const navItems = $derived.by(() =>
		(navMenus ?? []).filter((item) => item.name && item.url).slice(0, 6)
	);

	const titleVariantClassMap: Record<string, string> = {
		hero_h1_highlight: 'text-jade-600 dark:text-jade-300',
		hero_h1_primary: 'text-ink-900 dark:text-ink-100',
		hero_h1_light: 'font-light text-ink-800 dark:text-ink-200',
		hero_h1_medium_gap: 'mx-1 font-medium text-ink-900 dark:text-ink-100',
		hero_code_inline:
			'rounded-md border border-jade-500/25 bg-jade-50/80 px-2.5 py-1 font-mono text-[0.78em] font-semibold text-jade-700 shadow-subtle dark:border-jade-400/25 dark:bg-jade-400/10 dark:text-jade-200',
		hero_cursor:
			'inline-block h-[0.82em] w-px translate-y-[0.1em] bg-jade-500/70 dark:bg-jade-300/80'
	};

	function resolveTitleNodeClass(node: HomeHeroTemplateNode): string {
		const baseClass = node.type === 'code' ? '' : 'text-ink-800 dark:text-ink-200';
		const variantClass = node.variant ? (titleVariantClassMap[node.variant] ?? '') : '';
		const customClass = node.className ?? '';
		return `${baseClass} ${variantClass} ${customClass}`.trim();
	}
</script>

<section class="yohaku-first-viewport relative isolate min-h-[100svh] overflow-hidden">
	<div
		class="relative z-10 mx-auto flex min-h-[100svh] w-full max-w-7xl flex-col px-5 sm:px-8 lg:px-10"
	>
		<header class="relative flex min-h-20 items-center gap-4 py-4 sm:py-5">
			<a
				href={resolveHref('/')}
				class="group hidden min-w-0 items-center gap-3 text-ink-900 sm:flex dark:text-ink-100"
				aria-label={titleText}
			>
				<span
					class="flex h-10 w-10 shrink-0 items-center justify-center overflow-hidden rounded-full border border-ink-200 bg-white text-sm font-semibold text-ink-700 shadow-subtle dark:border-ink-700 dark:bg-ink-800 dark:text-ink-100"
				>
					{#if avatarUrl}
						<img src={avatarUrl} alt="" width="40" height="40" class="h-full w-full object-cover" />
					{:else}
						{siteInitial}
					{/if}
				</span>
			</a>

			{#if showTopNav && navItems.length > 0}
				<nav
					class="absolute left-1/2 top-1/2 hidden max-w-[68vw] -translate-x-1/2 -translate-y-1/2 items-center gap-1 rounded-default border border-ink-200/70 bg-white/55 p-1 shadow-subtle backdrop-blur-xl md:flex dark:border-ink-800/90 dark:bg-ink-900/60"
					aria-label="首页导航"
				>
					{#each navItems as item (item.id)}
						<a
							href={normalizeHref(item.url)}
							target={shouldOpenInNewTab(item.url) ? '_blank' : undefined}
							rel={shouldOpenInNewTab(item.url) ? 'noopener noreferrer' : undefined}
							class="flex h-9 items-center gap-2 rounded-default px-3 text-sm font-semibold transition-colors {isActiveNavItem(
								item.url
							)
								? 'bg-ink-900 text-white dark:bg-ink-100 dark:text-ink-950'
								: 'text-ink-600 hover:bg-ink-100 hover:text-ink-950 dark:text-ink-300 dark:hover:bg-ink-800 dark:hover:text-ink-50'}"
						>
							{#if item.icon}
								<DynamicLucideIcon name={item.icon} size={16} />
							{/if}
							<span>{item.name}</span>
						</a>
					{/each}
				</nav>
			{/if}

			<div class="ml-auto hidden sm:block">
				<div
					class="flex h-10 w-10 items-center justify-center rounded-full border border-ink-200/70 bg-white/50 text-ink-500 shadow-subtle dark:border-ink-800 dark:bg-ink-900/50 dark:text-ink-400"
					aria-hidden="true"
				>
					<UserRound size={16} />
				</div>
			</div>
		</header>

		<main class="flex flex-1 flex-col items-center justify-center py-10 text-center sm:py-12">
			<FadeIn y={18} duration={900}>
				<div class="flex flex-col items-center">
					<div
						class="mb-9 flex h-24 w-24 items-center justify-center overflow-hidden rounded-full border border-ink-200 bg-white text-3xl font-semibold text-ink-700 shadow-float dark:border-ink-700 dark:bg-ink-800 dark:text-ink-100"
					>
						{#if avatarUrl}
							<img
								src={avatarUrl}
								alt={titleText}
								width="96"
								height="96"
								fetchpriority="high"
								class="h-full w-full object-cover"
							/>
						{:else}
							{siteInitial}
						{/if}
					</div>

					<div
						class="yohaku-title max-w-5xl font-sans text-4xl font-medium leading-[1.22] text-ink-800 sm:text-5xl lg:text-6xl dark:text-ink-200"
					>
						{#each titleTemplate as node, idx (`${node.type}-${node.text ?? ''}-${idx}`)}
							{#if node.type === 'br'}
								<br />
							{:else if node.type === 'code'}
								<code class={resolveTitleNodeClass(node)}>{node.text ?? ''}</code>
							{:else}
								<span class={resolveTitleNodeClass(node)}>{node.text ?? ''}</span>
							{/if}
						{/each}
					</div>

					{#if description}
						<p
							class="mt-7 max-w-2xl text-xs font-medium uppercase leading-7 tracking-[0.16em] text-ink-500 sm:text-sm dark:text-ink-500"
						>
							{description}
						</p>
					{/if}

					<div class="mt-32 text-sm text-ink-400 dark:text-ink-500">
						{#if mottoLines.length > 0}
							<div class="font-serif italic">
								{#each mottoLines as line, index (`${line}-${index}`)}
									<span>{line}</span>{#if index < mottoLines.length - 1}<span class="mx-2">/</span
										>{/if}
								{/each}
							</div>
						{/if}
					</div>
				</div>
			</FadeIn>
		</main>

		{#if showTopNav && navItems.length > 0}
			<nav class="flex flex-wrap justify-center gap-2 pb-5 md:hidden" aria-label="首页导航">
				{#each navItems.slice(0, 4) as item (item.id)}
					<a
						href={normalizeHref(item.url)}
						target={shouldOpenInNewTab(item.url) ? '_blank' : undefined}
						rel={shouldOpenInNewTab(item.url) ? 'noopener noreferrer' : undefined}
						class="flex h-9 items-center gap-1.5 rounded-full border border-ink-200 bg-white/70 px-3 text-xs font-medium text-ink-600 dark:border-ink-700 dark:bg-ink-900/70 dark:text-ink-300"
					>
						{#if item.icon}
							<DynamicLucideIcon name={item.icon} size={13} />
						{/if}
						<span>{item.name}</span>
					</a>
				{/each}
			</nav>
		{/if}

		<FadeIn y={8} duration={700} delay={240} class="pb-8">
			<div class="flex flex-wrap justify-center gap-4">
				{#each socials as social, index (`${social.icon}-${social.href}-${index}`)}
					<a
						href={normalizeHref(social.href)}
						target={shouldOpenInNewTab(social.href) ? '_blank' : undefined}
						rel={shouldOpenInNewTab(social.href) ? 'noopener noreferrer' : undefined}
						title={social.name || social.icon}
						aria-label={social.name || social.icon}
						class="flex h-11 w-11 items-center justify-center rounded-full border border-ink-300/80 bg-white/35 text-ink-500 shadow-subtle backdrop-blur-sm transition-colors hover:border-jade-400 hover:bg-jade-50 hover:text-jade-700 dark:border-ink-700 dark:bg-ink-900/35 dark:text-ink-400 dark:hover:border-jade-500/70 dark:hover:bg-jade-400/10 dark:hover:text-jade-200"
					>
						<DynamicLucideIcon name={social.icon} size={16} />
					</a>
				{/each}
			</div>
		</FadeIn>
	</div>

	{#if showScrollHint}
		<a
			href="#home-content"
			class="absolute right-6 bottom-6 z-20 hidden h-11 w-11 items-center justify-center rounded-full border border-ink-200 bg-white/70 text-ink-400 shadow-subtle transition-colors hover:border-jade-300 hover:text-jade-600 md:flex dark:border-ink-700 dark:bg-ink-900/70 dark:text-ink-500 dark:hover:border-jade-700 dark:hover:text-jade-300"
			aria-label="滚动到内容"
		>
			<ArrowDown size={18} />
		</a>
	{/if}
</section>

<style lang="postcss">
	@reference "$routes/layout.css";

	.yohaku-first-viewport {
		background-color: var(--color-ink-50);
	}

	.yohaku-first-viewport::before {
		content: '';
		position: absolute;
		inset: 0;
		z-index: -2;
		background-image:
			linear-gradient(rgba(28, 25, 23, 0.035) 1px, transparent 1px),
			linear-gradient(90deg, rgba(28, 25, 23, 0.035) 1px, transparent 1px);
		background-size: 44px 44px;
		mask-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.65), transparent 78%);
	}

	.yohaku-first-viewport::after {
		content: '';
		position: absolute;
		inset: auto 0 0;
		z-index: -1;
		height: 34%;
		background: linear-gradient(to bottom, transparent, var(--color-ink-50));
	}

	:global(.dark) .yohaku-first-viewport {
		background-color: var(--color-ink-900);
	}

	:global(.dark) .yohaku-first-viewport::before {
		background-image:
			linear-gradient(rgba(245, 245, 244, 0.045) 1px, transparent 1px),
			linear-gradient(90deg, rgba(245, 245, 244, 0.045) 1px, transparent 1px);
	}

	:global(.dark) .yohaku-first-viewport::after {
		background: linear-gradient(to bottom, transparent, var(--color-ink-900));
	}
</style>
