<script lang="ts">
	import { onMount } from 'svelte';
	import {
		systemPrefersMode,
		setMode,
		localStorageKey,
		mode,
		themeColors as themeColorsStore,
		setInitialMode,
	} from './mode.js';
	import type { Mode, ThemeColors } from './types.js';
	import { isValidMode } from './stores.js';

	export let track = true;
	export let defaultMode: Mode = 'system';
	export let themeColors: ThemeColors = undefined;

	themeColorsStore.set(themeColors);

	onMount(() => {
		const unsubscriber = mode.subscribe(() => {});
		systemPrefersMode.tracking(track);
		systemPrefersMode.query();
		const localStorageMode = localStorage.getItem(localStorageKey);
		setMode(isValidMode(localStorageMode) ? localStorageMode : defaultMode);

		return () => {
			unsubscriber();
		};
	});

	const args = `"${defaultMode}"${themeColors ? `, ${JSON.stringify(themeColors)}` : ''}`;
</script>

<svelte:head>
	{#if themeColors}
		<!-- default to dark mode for to allow testing -->
		<!-- this will be overwritten by FOUC prevention snippet below -->
		<!-- but that snippet does not run in vitest -->
		<meta name="theme-color" content={themeColors.dark} />
	{/if}
	<!-- eslint-disable-next-line svelte/no-at-html-tags -->
	{@html `<script nonce="%sveltekit.nonce%">(` +
		setInitialMode.toString() +
		`)(` +
		args +
		`);</script>`}
</svelte:head>
