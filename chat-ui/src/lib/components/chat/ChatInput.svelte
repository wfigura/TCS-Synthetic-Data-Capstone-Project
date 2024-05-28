<script lang="ts">
	import { createEventDispatcher, onMount } from "svelte";
	import Papa from "papaparse";
	import CarbonUpload from "~icons/carbon/upload";

	export let value = "";
	export let minRows = 1;
	export let maxRows: null | number = null;
	export let placeholder = "";
	export let disabled = false;

	const TABLET_VIEWPORT_WIDTH = 768;

	let innerWidth = 0;
	let textareaElement: HTMLTextAreaElement;
	let isCompositionOn = false;

	const dispatch = createEventDispatcher<{ submit: { prompt: string; data: string } }>();

	$: minHeight = `${1 + minRows * 1.5}em`;
	$: maxHeight = maxRows ? `${1 + maxRows * 1.5}em` : `auto`;

	function handleKeydown(event: KeyboardEvent) {
		if (event.key === "Enter" && !event.shiftKey && !isCompositionOn) {
			event.preventDefault();
			textareaElement.blur();
			if (innerWidth > TABLET_VIEWPORT_WIDTH) {
				textareaElement.focus();
			}
			dispatch("submit", {
				prompt: "generate synthetic data for me based on the data I provided",
				data: value,
			});
		}
	}

	onMount(() => {
		if (innerWidth > TABLET_VIEWPORT_WIDTH) {
			textareaElement.focus();
		}
	});

	export let classNames = "";
	export let files: File[] = [];
	let filelist: FileList;
	let fileData: string = "";

	$: if (filelist) {
		files = Array.from(filelist);
		if (files.length > 0) {
			processCSV(files[0]);
		}
	}

	function processCSV(file: File) {
		const reader = new FileReader();
		reader.onload = (event) => {
			const csvData = event.target?.result as string;
			Papa.parse(csvData, {
				header: true,
				complete: (results: any) => {
					const filteredData = results.data.filter((row: any) => {
						return Object.values(row).some((value) => value.trim() !== "");
					});
					fileData = JSON.stringify(filteredData, null, 2);
					value = fileData; // Update the textarea value with filtered data
					console.log("Filtered CSV Data:", filteredData);
					// Trigger the submit event after processing CSV with the prompt
					dispatch("submit", {
						prompt: "generate synthetic data for me based on the data I provided",
						data: fileData,
					});
				},
			});
		};
		reader.readAsText(file);
	}
</script>

<svelte:window bind:innerWidth />

<div class="relative flex min-w-0 flex-1 items-center">
	<pre
		class="scrollbar-custom invisible overflow-x-hidden overflow-y-scroll whitespace-pre-wrap break-words"
		aria-hidden="true"
		style="min-height: {minHeight}; max-height: {maxHeight}">{" " + "\n"}</pre>

	<textarea
		enterkeyhint="send"
		tabindex="0"
		rows="1"
		class="scrollbar-custom m-0 h-full w-full flex-1 resize-none scroll-p-3 overflow-x-hidden overflow-y-scroll border-0 bg-transparent p-3 outline-none focus:ring-0 focus-visible:ring-0"
		class:text-gray-400={disabled}
		bind:this={textareaElement}
		{disabled}
		on:keydown={handleKeydown}
		on:compositionstart={() => (isCompositionOn = true)}
		on:compositionend={() => (isCompositionOn = false)}
		on:beforeinput
		{placeholder}
	/>
	<button
		class="btn relative h-8 rounded-lg border bg-white px-3 py-1 text-sm text-gray-500 shadow-sm transition-all hover:bg-gray-100 dark:border-gray-600 dark:bg-gray-700 dark:text-gray-300 dark:hover:bg-gray-600 {classNames}"
	>
		<input
			bind:files={filelist}
			class="absolute w-full cursor-pointer opacity-0"
			type="file"
			accept=".csv"
		/>
		<CarbonUpload class="mr-2 text-xs" /> Upload
	</button>
</div>

<style>
	pre,
	textarea {
		font-family: inherit;
		box-sizing: border-box;
		line-height: 1.5;
	}
	.relative.flex {
		align-items: center;
	}
</style>
