<script>
	export let showOnPx = 150;
	let hidden = true;
	function goTop() {
		// document.body.scrollIntoView();
		window.scrollTo({top: 0, behavior: 'smooth'});
	}
	function scrollContainer() {
		return document.documentElement || document.body;
	}
	function handleOnScroll() {
		if (!scrollContainer()) {
			return;
		}
		if (scrollContainer().scrollTop > showOnPx) {
			hidden = false;
		} else {
			hidden = true;
		}
	}
</script>

<svelte:window on:scroll={handleOnScroll} />
<button class="back-to-top" on:click={goTop} class:hidden>🔝</button>

<style>
	.back-to-top {
		opacity: 1;
		transition: opacity 0.5s, visibility 0.5s;
		position: fixed;
		z-index: 99;
		right: 20px;
		user-select: none;
		bottom: 20px;
		color: white;
		font-size: 28px;
		border: none;
		border-radius: 5px;
		padding: 0.05rem;
		width: 40px;
		height: 40px;
		cursor: pointer;
		background-color: lightgray;
	}
	.back-to-top.hidden {
		opacity: 0;
		visibility: hidden;
	}
</style>
