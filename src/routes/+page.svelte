<script>
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Svelte demo app" />
</svelte:head>

<section>
	<h1>
		SCRT
	</h1>

</section>

<style>
	section {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		flex: 1;
		background: var(--background);

		max-width: 100%;
		max-height: 100%;
		overflow: hidden;
	}

	h1 {
		width: 100%;
		color: transparent;
		font-family: var(--font-header);
		font-size: 40rem;
		line-height: 50%;
		margin: auto;
		padding-top: 15vh;
		-webkit-text-stroke: 1px #d0d0d0;
	}

</style>
