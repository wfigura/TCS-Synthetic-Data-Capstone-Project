<script lang="ts">
	import { onDestroy } from "svelte";
	import { goto, invalidate } from "$app/navigation";
	import { page } from "$app/stores";
	import "../styles/main.css";
	import { base } from "$app/paths";
	import {
		PUBLIC_APP_DESCRIPTION,
		PUBLIC_ORIGIN,
		PUBLIC_PLAUSIBLE_SCRIPT_URL,
	} from "$env/static/public";

	import { shareConversation } from "$lib/shareConversation";
	import { UrlDependency } from "$lib/types/UrlDependency";
	import { error } from "$lib/stores/errors";

	import Toast from "$lib/components/Toast.svelte";
	import { PUBLIC_APP_ASSETS, PUBLIC_APP_NAME } from "$env/static/public";
	import { createSettingsStore } from "$lib/stores/settings";
	import { browser } from "$app/environment";
	import DisclaimerModal from "$lib/components/DisclaimerModal.svelte";
	import TopNav from "$lib/components/TopNav.svelte";

	export let data;

	let errorToastTimeout: ReturnType<typeof setTimeout>;
	let currentError: string | null;

	async function onError() {
		// If a new different error comes, wait for the current error to hide first
		if ($error && currentError && $error !== currentError) {
			clearTimeout(errorToastTimeout);
			currentError = null;
			await new Promise((resolve) => setTimeout(resolve, 300));
		}

		currentError = $error;

		errorToastTimeout = setTimeout(() => {
			$error = null;
			currentError = null;
		}, 3000);
	}

	onDestroy(() => {
		clearTimeout(errorToastTimeout);
	});

	$: if ($error) onError();

	const settings = createSettingsStore(data.settings);

	$: if (browser && $page.url.searchParams.has("model")) {
		if ($settings.activeModel === $page.url.searchParams.get("model")) {
			goto(`${base}/?`);
		}
		$settings.activeModel = $page.url.searchParams.get("model") ?? $settings.activeModel;
	}
</script>

<svelte:head>
	<title>{PUBLIC_APP_NAME}</title>
	<meta name="description" content="The first open source alternative to ChatGPT. 💪" />
	<meta name="twitter:card" content="summary_large_image" />
	<meta name="twitter:site" content="@huggingface" />

	<!-- use those meta tags everywhere except on the share assistant page -->
	<!-- feel free to refacto if there's a better way -->
	{#if !$page.url.pathname.includes("/assistant/") && $page.route.id !== "/assistants"}
		<meta property="og:title" content={PUBLIC_APP_NAME} />
		<meta property="og:type" content="website" />
		<meta property="og:url" content="{PUBLIC_ORIGIN || $page.url.origin}{base}" />
		<meta
			property="og:image"
			content="{PUBLIC_ORIGIN || $page.url.origin}{base}/{PUBLIC_APP_ASSETS}/thumbnail.png"
		/>
		<meta property="og:description" content={PUBLIC_APP_DESCRIPTION} />
	{/if}
	<link
		rel="icon"
		href="{PUBLIC_ORIGIN || $page.url.origin}{base}/{PUBLIC_APP_ASSETS}/favicon.ico"
		sizes="32x32"
	/>
	<link
		rel="icon"
		href="{PUBLIC_ORIGIN || $page.url.origin}{base}/{PUBLIC_APP_ASSETS}/icon.svg"
		type="image/svg+xml"
	/>
	<link
		rel="apple-touch-icon"
		href="{PUBLIC_ORIGIN || $page.url.origin}{base}/{PUBLIC_APP_ASSETS}/apple-touch-icon.png"
	/>
	<link
		rel="manifest"
		href="{PUBLIC_ORIGIN || $page.url.origin}{base}/{PUBLIC_APP_ASSETS}/manifest.json"
	/>

	{#if PUBLIC_PLAUSIBLE_SCRIPT_URL && PUBLIC_ORIGIN}
		<script
			defer
			data-domain={new URL(PUBLIC_ORIGIN).hostname}
			src={PUBLIC_PLAUSIBLE_SCRIPT_URL}
		></script>
	{/if}
</svelte:head>

{#if !$settings.ethicsModalAccepted && $page.url.pathname !== `${base}/privacy`}
	<DisclaimerModal />
{/if}

<div class="text-smd grid h-full w-screen overflow-hidden dark:text-gray-300">
	{#if currentError}
		<Toast message={currentError} />
	{/if}
	<TopNav />
	<slot />
</div>
