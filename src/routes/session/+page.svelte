<script lang="ts">
	import { onMount } from 'svelte';
	import { base } from '$app/paths';
	import { page } from '$app/stores';
	import { browser } from '$app/environment';
	import { friendlyDates, sanitizeFilename } from '$lib/Constants.svelte';

	const searchParams = browser && $page.url.searchParams;
	let sessionId: string | null;
	if (searchParams) {
		sessionId = searchParams.get('id');
	}

	let loading = true;
	let sessionDetail: any;
	let eventDetail: any;
	let currentVideoTitle: string;
	let currentVideoUrl: string;

	let mediaList: {
		title: string;
		url: string;
		thumbnail: string;
		start: string;
	}[] = [];

	const fileExists = (fileUrl: string) => {
		var http = new XMLHttpRequest();

		http.open('HEAD', fileUrl, false);
		http.send();

		return http.status != 404;
	};

	const onClickPlaylistItem = (title: string, url: string) => {
		currentVideoTitle = title;
		currentVideoUrl = url;
	};

	const onDownload = (title: string, url: string) => {
		const anchor = document.createElement("a");
		anchor.href = url;

		const urlSplit = url.split(".");
		const fileExtension = "." + urlSplit[urlSplit.length - 1];
		anchor.download = sanitizeFilename(title) + fileExtension;
		// console.log("Downloaded file: " + anchor.download);

		anchor.target = "_blank";

		document.body.appendChild(anchor);
		anchor.click();
		document.body.removeChild(anchor);
	}

	onMount(async () => {
		// await new Promise(r => setTimeout(r, 3000));

		const response = await fetch(`${base}/json/${sessionId}.json`);
		sessionDetail = await response.json();
		eventDetail = sessionDetail.data.event;

		// Add publicly available media to the media list
		for (const item of eventDetail.schedule.items) {
			if (item.vod && item.vod.media && item.vod.media.element && item.vod.media.element.sources) {
				mediaList.push({
					title: item.title,
					url: item.vod.media.element.sources[0].uri,
					thumbnail: item.vod.media.thumbnail,
					start: item.start.split('T')[1].split('+')[0]
				});
			}
		}

		// Check and add hidden media
		if (eventDetail.picture) {
			// e.g. https://services.medicalcongress.online/congress/medias/2023/JFR-2023/2072/video/thumbs/poster.jpg
			const possibleVideoUrl =
				eventDetail.picture.split('/video/')[0] + '/video/y_1080p_4000kb.mp4';

			const urlAlreadyIncluded = mediaList.some((item) => item.url === possibleVideoUrl);
			if (!urlAlreadyIncluded && fileExists(possibleVideoUrl)) {
				mediaList.push({
					title: '[Unlisted Video]',
					url: possibleVideoUrl,
					thumbnail: eventDetail.picture,
					start: eventDetail.start.split('T')[1].split('+')[0]
				});
			}
		}

		if (mediaList.length > 0) {
			currentVideoTitle = mediaList[0].title;
			currentVideoUrl = mediaList[0].url;
		}

		loading = false;
	});
</script>

<svelte:head>
	<title>{eventDetail ? eventDetail.title : 'JFR 2023'}</title>
</svelte:head>

<main>
	<a href="{base}/" class="one-liner">
		<i class="gg-chevron-left" /> Retour
	</a>

	{#if loading}
		<div class="DNA_cont">
			<div class="nucleobase"></div>
			<div class="nucleobase"></div>
			<div class="nucleobase"></div>
			<div class="nucleobase"></div>
			<div class="nucleobase"></div>
			<div class="nucleobase"></div>
			<div class="nucleobase"></div>
			<div class="nucleobase"></div>
			<div class="nucleobase"></div>
			<div class="nucleobase"></div>
		</div>
	{/if}

	{#if !loading && eventDetail}
		<div class="detail">
			<div class="session-header">
				<h1>{eventDetail.title}</h1>
				<div class="date-time subtitle">
					<div>
						🗓️ {friendlyDates[eventDetail.start.split('T')[0]] ?? eventDetail.start.split('T')[0]}
					</div>
					<div>
						🕣 {eventDetail.start.split('T')[1].split('+')[0] +
							' - ' +
							eventDetail.end.split('T')[1].split('+')[0]}
					</div>
				</div>
			</div>
			<div class="objectives">
				<strong>Objectifs :</strong>
				<div>
					{@html eventDetail.objectives.join('')}
				</div>
			</div>
		</div>
		<div class="media">
			<div class="video-container">
				<div><strong>{currentVideoTitle}</strong></div>
				<video controls class="video-player" src={currentVideoUrl}>
					<track kind="captions" />
				</video>
			</div>
			<div class="playlist-container">
				<div><strong>Liste de lecture :</strong></div>
				<div class="playlist">
					{#each mediaList as item}
						<!-- svelte-ignore a11y-click-events-have-key-events -->
						<!-- svelte-ignore a11y-no-static-element-interactions -->
						<div
							class="playlist-item {item.url == currentVideoUrl ? 'selected' : ''}"
							on:click={() => onClickPlaylistItem(item.title, item.url)}
						>
							<div class="thumbnail-container">
								<img class="thumbnail" src={item.thumbnail} alt={item.title} />
							</div>
							<div class="video-details">
								<div><strong>{item.title}</strong></div>
								<div class="subtitle">
									<span>{item.start}</span>
									<button type="button" title="Télécharger" class="btn-download" on:click={() => onDownload(item.title, item.url)}>
										<svg
											width="16"
											height="16"
											viewBox="0 0 24 24"
											fill="none"
											class="svg-icon"
											xmlns="http://www.w3.org/2000/svg"
										>
											<path
												d="M11 5C11 4.44772 11.4477 4 12 4C12.5523 4 13 4.44772 13 5V12.1578L16.2428 8.91501L17.657 10.3292L12.0001 15.9861L6.34326 10.3292L7.75748 8.91501L11 12.1575V5Z"
												fill="currentColor"
											/>
											<path
												d="M4 14H6V18H18V14H20V18C20 19.1046 19.1046 20 18 20H6C4.89543 20 4 19.1046 4 18V14Z"
												fill="currentColor"
											/>
										</svg>
										<div>Télécharger</div>
									</button>
								</div>
							</div>
						</div>
					{/each}
				</div>
			</div>
		</div>
	{/if}
</main>

<style>
	* {
		box-sizing: border-box;
	}

	.one-liner {
		display: flex;
		align-items: center;
	}

	.gg-chevron-left {
		box-sizing: border-box;
		position: relative;
		display: block;
		transform: scale(var(--ggs, 1));
		width: 22px;
		height: 22px;
		border: 2px solid transparent;
		border-radius: 100px;
	}
	.gg-chevron-left::after {
		content: '';
		display: block;
		box-sizing: border-box;
		position: absolute;
		width: 10px;
		height: 10px;
		border-bottom: 2px solid;
		border-left: 2px solid;
		transform: rotate(45deg);
		left: 6px;
		top: 4px;
	}

	main {
		display: flex;
		flex-direction: column;
		align-items: flex-start;
		gap: 10px;
		padding: 15px;
		max-height: 100%;
		width: 100%;
	}

	.detail {
		display: flex;
		flex-direction: column;
		flex-grow: 0;
		gap: 10px;
	}

	.session-header {
		display: flex;
		flex-direction: column;
		align-items: baseline;
	}

	h1 {
		font-size: 20px;
	}

	.objectives {
		padding: 15px;
		background-color: rgb(24, 26, 27);
		border-radius: 5px;
	}

	.media {
		display: flex;
		gap: 15px;
		flex-grow: 1;
		width: 100%;
		overflow-y: hidden;
	}

	.video-container {
		display: flex;
		flex-direction: column;
		gap: 5px;
		flex-grow: 1;
		/* Rounded corners for video */
		border-radius: 5px;
		overflow: hidden;
		transform: translateZ(0);
		-webkit-transform: translateZ(0);
	}

	.playlist-container {
		flex: 0 0 35%;
		display: flex;
		flex-direction: column;
		gap: 10px;
		max-height: 100%;
	}

	.playlist {
		display: flex;
		flex-direction: column;
		gap: 10px;
		overflow-y: auto;
	}

	.playlist-item {
		display: flex;
		gap: 10px;
		border: 1px solid rgb(24, 26, 27);
		background-color: rgb(24, 26, 27);
		padding: 5px;
		border-radius: 5px;
		cursor: pointer;
	}

	.playlist-item.selected,
	.playlist-item:hover {
		border: 1px solid rgb(57, 62, 64);
		background-color: rgb(33, 36, 37);
	}

	.playlist-item.selected {
		border-left: 3px solid blueviolet;
	}

	.video-details {
		display: flex;
		flex-direction: column;
	}

	.thumbnail-container {
		display: flex;
		height: 100px;
		align-self: center;
	}

	.thumbnail {
		border-radius: 5px;
	}

	img {
		height: 100%;
		object-fit: contain;
	}

	video {
		margin: 0;
		max-width: 100%;
		height: calc(100% - 30px);
	}

	.btn-download {
		display: flex;
		align-items: center;
		border-radius: 5px;
		border: none;
		background-color: transparent;
		height: 24px;

		&:hover {
			color: white;
			background-color: #0077FF;
		}
	}

	@media (max-width: 1000px) {
		main {
			max-height: fit-content;
		}

		.objectives {
			width: 100%;
		}

		.media {
			flex-direction: column;
			overflow-y: auto;
		}

		.playlist {
			max-height: 300px;
		}

		.detail {
			width: 100%;
		}
	}

	@media (prefers-color-scheme: light) {
		.objectives {
			background-color: #e8e8e8;
		}

		.playlist-item {
			background-color: #e8e8e8;
			border-color: #e8e8e8;
		}

		.playlist-item.selected,
		.playlist-item:hover {
			border: 1px solid #bababa;
			background-color: #d1d1d1;
		}

		.playlist-item.selected {
			border-left: 3px solid blueviolet;
		}
	}
</style>
