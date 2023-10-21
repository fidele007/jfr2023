<script context="module">
    export const prerender = true;
</script>

<script lang="ts">
	import SessionCard from '$lib/SessionCard.svelte';
	import { onMount } from 'svelte';

	let selectedDate: string = '';
	let eventJson: any;
	let sessionsByDate: any = {};
	let filteredSessions: any;

	const searchSessions = (value: string) => {
		const scopedSessions = sessionsByDate[selectedDate];
		filteredSessions = value
			? scopedSessions.filter((x: any) => x.title.includes(value))
			: scopedSessions;
	};

	const onSearch = (event: Event) => {
		const value = (event.target as HTMLInputElement).value;
		searchSessions(value);
	};

	const onDateChange = (date: string) => {
		selectedDate = date;
		searchSessions((document.getElementById('search') as HTMLInputElement).value);
	};

	onMount(async () => {
		const response = await fetch('/json/event.json');
		eventJson = await response.json();

		sessionsByDate[''] = eventJson.data.event.search.items;
		sessionsByDate['2023-10-12'] = eventJson.data.event.search.items.filter((x: any) =>
			x.start.startsWith('2023-10-12')
		);
		sessionsByDate['2023-10-13'] = eventJson.data.event.search.items.filter((x: any) =>
			x.start.startsWith('2023-10-13')
		);
		sessionsByDate['2023-10-14'] = eventJson.data.event.search.items.filter((x: any) =>
			x.start.startsWith('2023-10-14')
		);
		sessionsByDate['2023-10-15'] = eventJson.data.event.search.items.filter((x: any) =>
			x.start.startsWith('2023-10-15')
		);
		sessionsByDate['2023-10-16'] = eventJson.data.event.search.items.filter((x: any) =>
			x.start.startsWith('2023-10-16')
		);

		searchSessions((document.getElementById('search') as HTMLInputElement).value);
	});
</script>

<svelte:head>
	<title>JFR 2023</title>
</svelte:head>

<main>
	<div>
		<img src="/jfr_2023.png" class="logo" alt="JFR Logo" />
		<h1>JFR 2023</h1>
	</div>
	<div id="search-container">
		<input id="search" type="text" placeholder="Search" on:input={onSearch} />
	</div>

	<ul id="menu">
		<li>
			<a
				href="/"
				class={selectedDate ? '' : 'selected'}
				on:click|preventDefault={() => onDateChange('')}>Tout</a
			>
		</li>
		|
		<li>
			<a
				href="/"
				class={selectedDate == '2023-10-12' ? 'selected' : ''}
				on:click|preventDefault={() => onDateChange('2023-10-12')}>jeu. 12 oct.</a
			>
		</li>
		|
		<li>
			<a
				href="/"
				class={selectedDate == '2023-10-13' ? 'selected' : ''}
				on:click|preventDefault={() => onDateChange('2023-10-13')}>ven. 13 oct.</a
			>
		</li>
		|
		<li>
			<a
				href="/"
				class={selectedDate == '2023-10-14' ? 'selected' : ''}
				on:click|preventDefault={() => onDateChange('2023-10-14')}>sam. 14 oct.</a
			>
		</li>
		|
		<li>
			<a
				href="/"
				class={selectedDate == '2023-10-15' ? 'selected' : ''}
				on:click|preventDefault={() => onDateChange('2023-10-15')}>dim. 15 oct.</a
			>
		</li>
		|
		<li>
			<a
				href="/"
				class={selectedDate == '2023-10-16' ? 'selected' : ''}
				on:click|preventDefault={() => onDateChange('2023-10-16')}>lun. 16 oct.</a
			>
		</li>
	</ul>

	<div class="card-container">
		{#if eventJson}
			{#each filteredSessions as item}
				<SessionCard info={item} />
			{/each}
		{/if}
	</div>
</main>

<style>
	main {
		max-width: 80%;
		min-width: 80%;
		margin: 0 auto;
		padding: 2rem;
		text-align: center;
	}

	.logo {
		height: 6em;
		will-change: filter;
		transition: filter 300ms;
	}
	.logo:hover {
		filter: drop-shadow(0 0 2em #646cffaa);
	}

	#search-container {
		margin-bottom: 1em;
	}

	#search-container input {
		min-width: 300px;
	}

	ul#menu li {
		display: inline;
	}

	a.selected {
		text-decoration: underline;
	}

	.card-container {
		display: flex;
		flex-direction: column;
		gap: 1rem;
	}
</style>
