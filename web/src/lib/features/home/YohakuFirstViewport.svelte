<script lang="ts">
	import { resolveHref } from '$lib/shared/utils/resolve-path';
	import { FadeIn } from '$lib/ui/animation';
	import DynamicLucideIcon from '$lib/ui/icons/DynamicLucideIcon.svelte';
	import { ArrowDown } from 'lucide-svelte';
	import type {
		HomeFirstViewportThemeConfig,
		HomeHeroSocialLink,
		HomeHeroTemplateNode,
		HomeHeroThemeConfig
	} from './types';

	let {
		config,
		firstViewport,
		siteName = 'Blog',
		fallbackImage = ''
	}: {
		config?: HomeHeroThemeConfig;
		firstViewport?: HomeFirstViewportThemeConfig;
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

	const splitCodeGlyphs = (text: string): Array<{ text: string; gapBefore: boolean }> => {
		const chars = Array.from(text);
		return chars.map((char, index) => ({
			text: char,
			gapBefore: char === '>' && chars[index - 1] === '/'
		}));
	};

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
	const showScrollHint = $derived(firstViewport?.showScrollHint ?? true);

	const titleVariantClassMap: Record<string, string> = {
		hero_h1_highlight: 'font-medium text-jade-600 dark:text-jade-300',
		hero_h1_primary: 'font-medium text-ink-900 dark:text-ink-100',
		hero_h1_light: 'font-light text-ink-800 dark:text-ink-200',
		hero_h1_medium_gap: 'mx-1 font-medium text-ink-900 dark:text-ink-100',
		hero_code_inline:
			'ml-2 rounded-lg border border-jade-500/18 bg-white/80 px-2.5 py-1 font-mono text-[0.76em] font-medium text-jade-700 shadow-subtle dark:border-jade-400/20 dark:bg-jade-400/10 dark:text-jade-200 sm:ml-3',
		hero_cursor:
			'home-hero-typewriter-cursor ml-1 inline-block font-mono font-light text-jade-500 dark:text-jade-300'
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
		<main class="flex flex-1 flex-col items-center justify-center py-24 text-center sm:py-28">
			<FadeIn y={18} duration={900}>
				<div class="flex flex-col items-center">
					<div
						class="mb-8 flex h-22 w-22 items-center justify-center overflow-hidden rounded-full border border-ink-200/80 bg-white/90 text-3xl font-semibold text-ink-700 shadow-subtle dark:border-ink-700/80 dark:bg-ink-800 dark:text-ink-100 sm:h-24 sm:w-24"
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
						class="yohaku-title max-w-4xl font-sans text-[2rem] font-medium leading-[1.3] text-ink-800 sm:text-[2.65rem] sm:leading-[1.24] lg:text-[2.95rem] dark:text-ink-200"
					>
						{#each titleTemplate as node, idx (`${node.type}-${node.text ?? ''}-${idx}`)}
							{#if node.type === 'br'}
								<br />
							{:else if node.type === 'code'}
								<code class={resolveTitleNodeClass(node)}
									>{#each splitCodeGlyphs(node.text ?? '') as glyph, glyphIdx (`${glyph.text}-${glyphIdx}`)}<span
											class:home-hero-code-glyph-gap={glyph.gapBefore}>{glyph.text}</span
										>{/each}</code
								>
							{:else}
								<span class={resolveTitleNodeClass(node)}>{node.text ?? ''}</span>
							{/if}
						{/each}
					</div>

					{#if description}
						<p
							class="mt-7 max-w-2xl text-xs font-medium uppercase leading-7 tracking-[0.14em] text-ink-500 sm:text-[13px] dark:text-ink-500"
						>
							{description}
						</p>
					{/if}

					<div class="mt-24 text-sm text-ink-400 dark:text-ink-500 sm:mt-28">
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

		<FadeIn y={8} duration={700} delay={240} class="pb-8">
			<div class="flex flex-wrap justify-center gap-3">
				{#each socials as social, index (`${social.icon}-${social.href}-${index}`)}
					<a
						href={normalizeHref(social.href)}
						target={shouldOpenInNewTab(social.href) ? '_blank' : undefined}
						rel={shouldOpenInNewTab(social.href) ? 'noopener noreferrer' : undefined}
						title={social.name || social.icon}
						aria-label={social.name || social.icon}
						class="flex h-10 w-10 items-center justify-center rounded-full border border-ink-300/70 bg-white/45 text-ink-500 shadow-subtle backdrop-blur-sm transition-all duration-200 hover:-translate-y-0.5 hover:border-jade-400 hover:bg-jade-50 hover:text-jade-700 dark:border-ink-700 dark:bg-ink-900/35 dark:text-ink-400 dark:hover:border-jade-500/70 dark:hover:bg-jade-400/10 dark:hover:text-jade-200"
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
		background:
			linear-gradient(to bottom, rgba(255, 255, 255, 0.72), rgba(255, 255, 255, 0)),
			var(--texture-noise);
		background-size:
			auto,
			180px 180px;
		opacity: 0.36;
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
		background:
			linear-gradient(to bottom, rgba(12, 10, 9, 0.52), rgba(12, 10, 9, 0)),
			var(--texture-noise);
		opacity: 0.18;
	}

	:global(.dark) .yohaku-first-viewport::after {
		background: linear-gradient(to bottom, transparent, var(--color-ink-900));
	}

	:global(.home-hero-typewriter-cursor) {
		animation: home-hero-typewriter-cursor 2.4s ease-in-out infinite;
	}

	@keyframes -global-home-hero-typewriter-cursor {
		0%,
		100% {
			opacity: 1;
		}
		55% {
			opacity: 0.18;
		}
	}

	.home-hero-code-glyph-gap {
		margin-left: 0.12em;
	}
</style>
