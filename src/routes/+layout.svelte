<script>
	import "@fontsource-variable/geist"
	import "../app.css";
	import './layout.css';
	import lightFavicon from '#lib/assets/logo/ico_rvb_cbg.svg';
	import darkFavicon from '#lib/assets/logo/ico_rvb_dbg.svg';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" type="image/svg+xml" href={lightFavicon} media="(prefers-color-scheme: light)" />
	<link rel="icon" type="image/svg+xml" href={darkFavicon} media="(prefers-color-scheme: dark)" />
	<title>OSTRA | Online document signer</title>
</svelte:head>

{@render children()}
