<script lang="ts">
	import SocialItem from '$lib/features/home/SocialItem.svelte';
	import { FadeIn } from '$lib/ui/animation';
	import { ArrowDown } from 'lucide-svelte';
	import type { HomeHeroSocialLink, HomeHeroTemplateNode, HomeHeroThemeConfig } from './types';

	let { config }: { config?: HomeHeroThemeConfig } = $props();

	const defaultTitleTemplate: HomeHeroTemplateNode[] = [
		{ type: 'h1', text: 'Hi! 👋', variant: 'hero_h1_highlight' },
		{ type: 'br' },
		{ type: 'h1', text: "I'm grtsinry43", variant: 'hero_h1_primary' }
	];
	const defaultDescription =
		'Java & JavaScript full-stack developer committed to crafting excellent software.';
	const defaultAvatarUrl = '';
	const defaultMottoLines = [
		'热衷于在逻辑与感性的缝隙中构建数字花园。',
		'也许，代码是现代的诗歌，而文字是思想的快照。'
	];
	const defaultSocials: HomeHeroSocialLink[] = [
		{ icon: 'github', name: 'GitHub', href: 'https://github.com/grtinry43' },
		{ icon: 'mail', name: 'Email', href: 'mailto:grtsinry43@outlook.com' },
		{ icon: 'rss', name: 'RSS', href: '/feed' }
	];

	const variantClassMap: Record<string, string> = {
		hero_h1_highlight: 'italic text-jade-600 dark:text-jade-400 font-light text-4xl lg:text-5xl',
		hero_h1_primary: 'text-ink-900 dark:text-ink-100 font-semibold text-4xl lg:text-5xl',
		hero_h1_light: 'font-light text-4xl lg:text-5xl text-ink-900 dark:text-ink-100',
		hero_h1_medium_gap: 'font-medium mx-2 text-4xl lg:text-5xl text-ink-900 dark:text-ink-100',
		hero_code_inline:
			'font-medium mx-2 rounded-md border border-ink-200/70 bg-white/50 px-2 py-0.5 text-[0.82em] text-ink-800 shadow-subtle transition-colors duration-200 hover:border-jade-300 hover:bg-jade-50/70 hover:text-jade-700 dark:border-ink-700/70 dark:bg-ink-800/20 dark:text-ink-100 dark:hover:border-jade-600/60 dark:hover:bg-jade-400/10 dark:hover:text-jade-200',
		hero_cursor:
			'home-hero-typewriter-cursor ml-1 inline-block font-mono font-light text-jade-500 dark:text-jade-300'
	};

	const titleTemplate = $derived(
		config?.titleTemplate && config.titleTemplate.length > 0
			? config.titleTemplate
			: defaultTitleTemplate
	);
	const description = $derived(config?.description || defaultDescription);
	const avatarUrl = $derived(config?.avatarUrl || defaultAvatarUrl);
	const mottoLines = $derived(
		config?.mottoLines && config.mottoLines.length > 0 ? config.mottoLines : defaultMottoLines
	);
	const mottoLinesAlign = $derived(config?.mottoLinesAlign ?? 'default');
	const socials = $derived(
		config?.socials && config.socials.length > 0 ? config.socials : defaultSocials
	);
	const socialsAlign = $derived(config?.socialsAlign ?? 'default');

	const splitCodeGlyphs = (text: string): Array<{ text: string; gapBefore: boolean }> => {
		const chars = Array.from(text);
		return chars.map((char, index) => ({
			text: char,
			gapBefore: char === '>' && chars[index - 1] === '/'
		}));
	};

	function resolveNodeClass(node: HomeHeroTemplateNode): string {
		const baseClass = node.type === 'h1' ? 'text-ink-900 dark:text-ink-100' : '';
		const variantClass = node.variant ? (variantClassMap[node.variant] ?? '') : '';
		const customClass = node.className ?? '';
		return `${baseClass} ${variantClass} ${customClass}`.trim();
	}
</script>

<div
	class="hero-container min-h-[calc(100svh-5rem)] md:min-h-[calc(100svh-7rem)] flex w-full flex-col justify-center"
>
	<!-- [Desktop Version] -->
	<div class="mx-auto hidden w-full max-w-6xl flex-col gap-16 px-8 md:flex">
		<div class="hero-info flex items-center justify-center gap-16 lg:gap-24">
			{#if avatarUrl}
				<FadeIn y={24} duration={1000}>
					<div class="hero-author-avatar relative z-10 w-fit">
						<div
							class="absolute inset-0 translate-x-3 translate-y-3 rounded-default border border-ink-200/80 dark:border-ink-800"
						></div>
						<img
							src={avatarUrl}
							alt="Author"
							width="192"
							height="192"
							fetchpriority="high"
							class="relative h-48 w-48 rounded-default object-cover shadow-float ring-1 ring-ink-200 dark:ring-ink-700"
						/>
					</div>
				</FadeIn>
			{/if}
			<FadeIn y={20} duration={1000} delay={200}>
				<div class="hero-welcome group">
					<div class="hero-title-desktop font-mono leading-[1.45]">
						{#each titleTemplate as node, idx (`${node.type}-${node.text ?? ''}-${idx}`)}
							{#if node.type === 'br'}
								<br />
								<div class="mt-3"></div>
							{:else if node.type === 'code'}
								<code class={resolveNodeClass(node)}
									>{#each splitCodeGlyphs(node.text ?? '') as glyph, glyphIdx (`desktop-${glyph.text}-${glyphIdx}`)}<span
											class:home-hero-code-glyph-gap={glyph.gapBefore}>{glyph.text}</span
										>{/each}</code
								>
							{:else if node.type === 'span'}
								<span class={resolveNodeClass(node)}>{node.text ?? ''}</span>
							{:else}
								<h1 class={resolveNodeClass(node)}>{node.text ?? ''}</h1>
							{/if}
						{/each}
					</div>
					<p class="hero-subtitle mt-8 max-w-xl font-mono text-sm leading-7 text-ink-500">
						{description}
					</p>
				</div>
			</FadeIn>
		</div>

		<div class="mx-auto flex w-full max-w-5xl flex-col gap-10">
			<FadeIn y={16} duration={900} delay={400}>
				<div
					class="hero-motto font-serif text-[1.45rem] leading-loose text-ink-800 dark:text-ink-200"
					class:text-center={mottoLinesAlign === 'center'}
				>
					{#each mottoLines as line, lineIdx (`${line}-${lineIdx}`)}
						{line}<br />
					{/each}
				</div>
			</FadeIn>

			<FadeIn y={12} duration={800} delay={600}>
				<div
					class="social-container flex items-center gap-5"
					class:justify-center={socialsAlign === 'center'}
				>
					{#each socials as social, socialIdx (`${social.icon}-${social.href}-${socialIdx}`)}
						<SocialItem icon={social.icon} name={social.name} href={social.href} />
					{/each}
				</div>
			</FadeIn>
		</div>
	</div>

	<!-- [Mobile Version] -->
	<div class="flex flex-col items-center pt-8 md:hidden">
		{#if avatarUrl}
			<FadeIn y={15} duration={1000}>
				<div class="relative mb-10">
					<div
						class="absolute inset-0 translate-x-2 translate-y-2 border border-ink-200 dark:border-ink-800 rounded-default -z-10"
					></div>
					<img
						src={avatarUrl}
						alt="Author"
						width="110"
						height="110"
						fetchpriority="high"
						class="h-[110px] w-[110px] rounded-default object-cover ring-1 ring-ink-100 dark:ring-ink-800 shadow-sm"
					/>
				</div>
			</FadeIn>
		{/if}

		<FadeIn y={10} duration={1000} delay={200}>
			<div class="text-center px-6 group">
				<div class="hero-title-mobile font-mono leading-relaxed tracking-normal">
					{#each titleTemplate as node, idx (`mobile-${node.type}-${node.text ?? ''}-${idx}`)}
						{#if node.type === 'br'}
							<br />
						{:else if node.type === 'code'}
							<code class={resolveNodeClass(node)}
								>{#each splitCodeGlyphs(node.text ?? '') as glyph, glyphIdx (`mobile-${glyph.text}-${glyphIdx}`)}<span
										class:home-hero-code-glyph-gap={glyph.gapBefore}>{glyph.text}</span
									>{/each}</code
							>
						{:else if node.type === 'span'}
							<span class={resolveNodeClass(node)}>{node.text ?? ''}</span>
						{:else}
							<h1 class={resolveNodeClass(node)}>{node.text ?? ''}</h1>
						{/if}
					{/each}
				</div>
				<p class="mt-5 text-[11px] font-mono leading-relaxed text-ink-500">
					{description}
				</p>
			</div>
		</FadeIn>

		<FadeIn y={8} duration={800} delay={400} class="mt-12">
			<div class="flex items-center gap-5">
				{#each socials as social, socialIdx (`mobile-${social.icon}-${social.href}-${socialIdx}`)}
					<SocialItem icon={social.icon} name="" href={social.href} />
					{#if socialIdx < socials.length - 1}
						<span class="w-px h-3 bg-ink-200 dark:bg-ink-800"></span>
					{/if}
				{/each}
			</div>
		</FadeIn>
	</div>

	<div class="hero-scroll-hint hidden md:flex" aria-hidden="true">
		<ArrowDown size={20} />
	</div>
</div>

<style lang="postcss">
	@reference "$routes/layout.css";

	.hero-container {
		@apply relative;
	}

	.hero-scroll-hint {
		@apply absolute right-10 bottom-8 flex h-12 w-12 items-center justify-center text-ink-400 opacity-40;
		animation: hero-scroll-bounce 1.6s ease-in-out infinite;
	}

	@keyframes hero-scroll-bounce {
		0%,
		100% {
			transform: translateY(0);
		}
		50% {
			transform: translateY(8px);
		}
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

	:global(.hero-title-desktop h1) {
		display: inline;
	}

	:global(.hero-title-mobile h1) {
		display: inline;
		font-size: 1.25rem;
		font-weight: 700;
	}

	:global(.hero-title-mobile code) {
		font-size: 1rem;
	}

	/* 特别为移动端 SocialItem 去掉文字 */
	:global(.md\:hidden .social-container span) {
		display: none;
	}
</style>
