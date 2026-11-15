<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';
	import { onMount } from 'svelte';
	import Lenis from 'lenis';
	import 'lenis/dist/lenis.css';
	import type { LayoutProps } from './$types';

	let { children }: LayoutProps = $props();

	onMount(() => {
		// Kalau sistem diminta minim gerakan, biarkan browser yang mengurus
		// scroll. Scroll halus yang dipaksakan bisa bikin pusing.
		if (matchMedia('(prefers-reduced-motion: reduce)').matches) return;

		// `anchors: true` wajib. Tanpa ini, Lenis justru mematikan link #proyek.
		const lenis = new Lenis({ anchors: true });

		let frame = requestAnimationFrame(function raf(time) {
			lenis.raf(time);
			frame = requestAnimationFrame(raf);
		});

		return () => {
			cancelAnimationFrame(frame);
			lenis.destroy();
		};
	});
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
	<title>Portofolio — OOTS 2026</title>
</svelte:head>

{@render children()}
