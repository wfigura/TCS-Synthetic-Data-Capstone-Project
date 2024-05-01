<script lang="ts">
	import { onDestroy } from "svelte";

	import IconCopy from "./icons/IconCopy.svelte";
	import Tooltip from "./Tooltip.svelte";

	export let classNames = "";
	export let value: string;

	let isSuccess = false;
	let timeout: ReturnType<typeof setTimeout>;

	const handleClick = async () => {
		// writeText() can be unavailable or fail in some cases (iframe, etc) so we try/catch
		// Prepare the file data (e.g., content or URL)
        const fileData = value; // File content or URL
        // Create a Blob object from the file data
        const blob = new Blob([fileData], { type: 'application/octet-stream' });
        // Generate a URL for the Blob object
        const url = URL.createObjectURL(blob);
        // Create an anchor element
        const anchor = document.createElement('a');
        // Set the href attribute to the generated URL
        anchor.href = url;
        // Set the download attribute with the desired file name
        anchor.download = 'sampleData.csv'; // Set the file name here
        // Simulate a click on the anchor element to trigger the download
        anchor.click();
        // Clean up by revoking the object URL
        URL.revokeObjectURL(url);
	};

	onDestroy(() => {
		if (timeout) {
			clearTimeout(timeout);
		}
	});
</script>

<button
	class="btn rounded-lg border border-gray-200 px-2 py-2 text-sm shadow-sm transition-all hover:border-gray-300 active:shadow-inner dark:border-gray-700 dark:hover:border-gray-500 {classNames}"
	title={"Download File"}
	type="button"
	on:click
	on:click={handleClick}
>
	<div class="relative">
		<slot>
			Download
		</slot>

		<Tooltip classNames={isSuccess ? "opacity-100" : "opacity-0"} />
	</div>
</button>
