<script lang="ts">
	import { page } from '$app/state';
	import { resolveHref } from '$lib/shared/utils/resolve-path';
	import { uiState } from '$lib/shared/stores/ui.svelte';
	import DynamicLucideIcon from '$lib/ui/icons/DynamicLucideIcon.svelte';
	import ThemeIcon from '$lib/ui/layout/sidebar/ThemeIcon.svelte';
	import VisitorAvatar from '$lib/ui/layout/sidebar/VisitorAvatar.svelte';
	import { Search } from 'lucide-svelte';
	import type { NavMenuItem } from '$lib/features/navigation/types';

	let {
		menuTree = [],
		avatarUrl = '',
		siteName = 'Blog',
		showNav = true
	}: {
		menuTree?: NavMenuItem[];
		avatarUrl?: string;
		siteName?: string;
		showNav?: boolean;
	} = $props();

	const isExternalHref = (href: string): boolean => /^(https?:|\/\/)/i.test(href);
	const normalizeHref = (href: string): string => (href.startsWith('/') ? resolveHref(href) : href);
	const normalizePathname = (href: string): string | null => {
		if (!href || isExternalHref(href)) return null;
		const path = href.startsWith('/') ? href : `/${href}`;
		return path.replace(/\/+$/, '') || '/';
	};

	const isActiveNavItem = (href: string): boolean => {
		const target = normalizePathname(href);
		if (!target) return false;
		const pathname = page.url.pathname.replace(/\/+$/, '') || '/';
		return target === '/' ? pathname === '/' : pathname === target || pathname.startsWith(`${target}/`);
	};

	const navItems = $derived.by(() =>
		(menuTree ?? []).filter((item) => item.name && item.url).slice(0, 7)
	);
	const siteInitial = $derived(siteName.trim().slice(0, 1).toUpperCase() || 'B');
</script>

<header class="yohaku-top-nav fixed inset-x-0 top-0 z-50 px-4 pt-3 sm:px-6 lg:px-10">
	<div class="relative mx-auto flex h-14 max-w-7xl items-center justify-between">
		<a
			href={resolveHref('/')}
			class="group flex min-w-0 items-center gap-3 text-ink-900 dark:text-ink-100"
			aria-label={siteName}
		>
			<span
				class="flex h-10 w-10 shrink-0 items-center justify-center overflow-hidden rounded-full border border-ink-200/80 bg-white/80 text-sm font-semibold text-ink-700 shadow-subtle backdrop-blur-md transition-transform duration-300 group-hover:-translate-y-0.5 dark:border-ink-700/80 dark:bg-ink-900/70 dark:text-ink-100"
			>
				{#if avatarUrl}
					<img src={avatarUrl} alt="" width="40" height="40" class="h-full w-full object-cover" />
				{:else}
					{siteInitial}
				{/if}
			</span>
			<span class="hidden max-w-44 truncate text-sm font-medium tracking-normal sm:block">
				{siteName}
			</span>
		</a>

		{#if showNav && navItems.length > 0}
			<nav
				class="yohaku-nav-pill absolute left-1/2 top-1/2 hidden max-w-[58vw] -translate-x-1/2 -translate-y-1/2 items-center gap-1 rounded-full border border-ink-200/65 bg-white/72 p-1 shadow-subtle backdrop-blur-xl md:flex dark:border-ink-800/85 dark:bg-ink-900/72"
				aria-label="站点导航"
			>
				{#each navItems as item (item.id)}
					<a
						href={normalizeHref(item.url)}
						target={isExternalHref(item.url) ? '_blank' : undefined}
						rel={isExternalHref(item.url) ? 'noopener noreferrer' : undefined}
						class="flex h-9 items-center gap-2 rounded-full px-3.5 text-sm font-medium transition-all duration-200 {isActiveNavItem(
							item.url
						)
							? 'bg-ink-900 text-white shadow-sm dark:bg-ink-100 dark:text-ink-950'
							: 'text-ink-500 hover:bg-ink-100/80 hover:text-ink-950 dark:text-ink-400 dark:hover:bg-ink-800/80 dark:hover:text-ink-50'}"
					>
						{#if item.icon}
							<DynamicLucideIcon name={item.icon} size={15} />
						{/if}
						<span>{item.name}</span>
					</a>
				{/each}
			</nav>
		{/if}

		<div
			class="flex h-11 items-center gap-1 rounded-full border border-ink-200/65 bg-white/70 p-0.5 shadow-subtle backdrop-blur-xl dark:border-ink-800/80 dark:bg-ink-900/70"
		>
			<button
				type="button"
				onclick={() => uiState.openSearch()}
				class="flex h-10 w-10 items-center justify-center rounded-full text-ink-400 transition-colors hover:bg-ink-100 hover:text-ink-900 dark:hover:bg-ink-800 dark:hover:text-ink-100"
				aria-label="搜索"
				title="搜索"
			>
				<Search size={18} />
			</button>
			<ThemeIcon />
			<VisitorAvatar />
		</div>
	</div>

	{#if showNav && navItems.length > 0}
		<nav class="mt-2 flex gap-2 overflow-x-auto pb-1 md:hidden" aria-label="站点导航">
			{#each navItems as item (item.id)}
				<a
					href={normalizeHref(item.url)}
					target={isExternalHref(item.url) ? '_blank' : undefined}
					rel={isExternalHref(item.url) ? 'noopener noreferrer' : undefined}
					class="flex h-9 shrink-0 items-center gap-1.5 rounded-full border px-3 text-xs font-medium transition-colors {isActiveNavItem(
						item.url
					)
						? 'border-ink-900 bg-ink-900 text-white dark:border-ink-100 dark:bg-ink-100 dark:text-ink-950'
						: 'border-ink-200/70 bg-white/68 text-ink-500 backdrop-blur-xl hover:text-ink-900 dark:border-ink-800 dark:bg-ink-900/70 dark:text-ink-400 dark:hover:text-ink-100'}"
				>
					{#if item.icon}
						<DynamicLucideIcon name={item.icon} size={13} />
					{/if}
					<span>{item.name}</span>
				</a>
			{/each}
		</nav>
	{/if}
</header>

<style lang="postcss">
	@reference "$routes/layout.css";

	.yohaku-top-nav {
		pointer-events: none;
	}

	.yohaku-top-nav > * {
		pointer-events: auto;
	}

	:global(.yohaku-top-nav [data-theme]),
	:global(.yohaku-top-nav [aria-label='登录']),
	:global(.yohaku-top-nav [aria-label='用户中心']) {
		border-radius: 9999px;
	}

	.yohaku-nav-pill {
		box-shadow:
			0 8px 30px -22px rgba(28, 25, 23, 0.38),
			inset 0 1px 0 rgba(255, 255, 255, 0.7);
	}

	:global(.dark) .yohaku-nav-pill {
		box-shadow:
			0 14px 36px -26px rgba(0, 0, 0, 0.8),
			inset 0 1px 0 rgba(255, 255, 255, 0.06);
	}
</style>
