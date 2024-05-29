<script lang="ts">
	import { createEventDispatcher, onMount } from "svelte";
	import UploadBtn from "../UploadBtn.svelte";
	import Papa from "papaparse";
	import type { Message } from "$lib/types/Message";

	export let value = "";
	export let minRows = 1;
	export let maxRows: null | number = null;
	export let placeholder = "";
	export let disabled = false;
	export let files: File[] = [];

	const TABLET_VIEWPORT_WIDTH = 768;

	let innerWidth = 0;
	let textareaElement: HTMLTextAreaElement;
	let isCompositionOn = false;

	const dispatch = createEventDispatcher<{ submit: void }>();

	$: minHeight = `${1 + minRows * 1.5}em`;
	$: maxHeight = maxRows ? `${1 + maxRows * 1.5}em` : `auto`;

	function handleKeydown(event: KeyboardEvent) {
		if (event.key === "Enter" && !event.shiftKey && !isCompositionOn) {
			event.preventDefault();
			textareaElement.blur();
			if (innerWidth > TABLET_VIEWPORT_WIDTH) {
				textareaElement.focus();
			}
			dispatch("submit");
		}
	}

	const dispatch2 = createEventDispatcher<{
		message: string;
		share: void;
		stop: void;
		retry: { id: Message["id"]; content?: string };
		continue: { id: Message["id"] };
		fileData: string;
	}>();

	let file;
	let firstFewRows = [];

	function handleFileUpload(event) {
		console.log("here???");
		file = event.target.files[0];
		if (file) {
			const reader = new FileReader();
			reader.onload = function (e) {
				const text = e.target.result;
				const parsedData = Papa.parse(text, { header: true });
				firstFewRows = parsedData.data.slice(0, 5);
				console.log(firstFewRows);
				let message = `Generate an additional 20 rows of new, unique data based on file ${files[0].name}. A preview of the file uploaded: ${firstFewRows}`;
				dispatch2("message", message);
			};
			reader.readAsText(file);
		}
	}

	let filelist: FileList;

	$: if (filelist) {
		files = Array.from(filelist);
	}

	onMount(() => {
		if (innerWidth > TABLET_VIEWPORT_WIDTH) {
			textareaElement.focus();
		}
	});
</script>

<svelte:window bind:innerWidth />

<div class="relative flex min-w-0 flex-1 items-center">
	<pre
		class="scrollbar-custom invisible overflow-x-hidden overflow-y-scroll whitespace-pre-wrap break-words"
		aria-hidden="true"
		style="min-height: {minHeight}; max-height: {maxHeight}"
	/>
	<!-- <UploadBtn bind:files on:change={handleFileUpload} /> -->
	<!-- <input type="file" bind:files={filelist} accept=".csv" on:change={handleFileUpload} /> -->

	<textarea
		enterkeyhint="send"
		tabindex="0"
		rows="1"
		class="scrollbar-custom m-0 h-full w-full flex-1 resize-none scroll-p-3 overflow-x-hidden overflow-y-scroll border-0 bg-transparent p-3 outline-none focus:ring-0 focus-visible:ring-0"
		class:text-gray-400={disabled}
		bind:value
		bind:this={textareaElement}
		{disabled}
		on:keydown={handleKeydown}
		on:compositionstart={() => (isCompositionOn = true)}
		on:compositionend={() => (isCompositionOn = false)}
		on:beforeinput
		{placeholder}
	/>
</div>

<style>
	pre,
	textarea {
		font-family: inherit;
		box-sizing: border-box;
		line-height: 1.8;
	}
	.relative.flex {
		align-items: center;
	}
</style>
