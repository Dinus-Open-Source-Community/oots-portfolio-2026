# CP 07 — Deploy

Halaman yang bisa dibuka orang lain tanpa laptop kalian menyala.

Checkpoint terakhir. Isinya dua hal: memindahkan yang sudah jadi ke Vercel, dan
tiga sentuhan akhir yang membuat halaman terasa selesai.

## Apa yang berubah

- `@sveltejs/adapter-vercel` menggantikan `adapter-auto`. `auto` menebak
  platform, `vercel` memilih dengan sadar.
- **Scroll halus** lewat Lenis, dipasang sekali di `+layout.svelte`.
- **Menu mobile** yang benar-benar terbuka. Sebelumnya tombolnya cuma ikon.
- **Form kontak memenuhi lebar induknya** di desktop (dari 440px jadi sisa
  ruang yang ada), dan tombol kirim ikut selebar form.
- **Isi diganti dengan milik sendiri** — nama, foto sampul, foto profil, dan
  email di footer bukan lagi placeholder. Ini langkah terakhir yang diminta
  README: "ganti nama, tagline, tiga proyek, dan foto dengan milik kalian".
  Karena tiap tema sekarang memakai satu foto yang sama, pasangan foto
  terang/gelap dari CP02 tidak dipakai lagi di branch ini. Struktur `theme`
  tetap ada; yang hilang hanya contoh pertukaran gambarnya.

## Salin ini

```sh
bun add -d @sveltejs/adapter-vercel
bun add lenis
```

`vite.config.ts`

```ts
import tailwindcss from '@tailwindcss/vite';
import adapter from '@sveltejs/adapter-vercel';
import { sveltekit } from '@sveltejs/kit/vite';
import { fileURLToPath } from 'node:url';
import { defineConfig } from 'vite';

export default defineConfig({
	resolve: {
		alias: { $lib: fileURLToPath(new URL('./src/lib', import.meta.url)) }
	},
	plugins: [
		tailwindcss(),
		sveltekit({
			compilerOptions: {
				// Force runes mode for the project, except for libraries. Can be removed in svelte 6.
				runes: ({ filename }) =>
					filename.split(/[/\\]/).includes('node_modules') ? undefined : true
			},

			// Adapter menentukan bentuk hasil build. Vercel butuh adapter
			// sendiri; `adapter-auto` hanya menebak dan tidak selalu benar.
			adapter: adapter()
		})
	]
});
```

`src/routes/+layout.svelte`

```svelte
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
```

`src/routes/+page.svelte`

```svelte
<script lang="ts">
	import { enhance } from '$app/forms';
	import { projects, profile } from '$lib/data';
	import ProjectRow from '$lib/components/ProjectRow.svelte';

	type Theme = 'light' | 'dark';
	type Status = 'idle' | 'loading' | 'success' | 'error';

	let theme: Theme = $state('light');
	let status: Status = $state('idle');
	let menuOpen: boolean = $state(false);
</script>

<svelte:window
	onkeydown={(event) => {
		if (event.key === 'Escape') menuOpen = false;
	}}
/>

<div class="min-h-dvh bg-bg text-ink {theme === 'dark' ? 'dark' : ''}">
	<!-- nav -->
	<header class="relative border-b border-line">
		<div
			class="mx-auto flex h-[60px] max-w-[1440px] items-center justify-between px-6 lg:h-[72px] lg:px-24"
		>
			<a href="/" class="text-[15px] font-semibold lg:text-base">dimas.avrian</a>

			<div class="flex items-center gap-2 lg:gap-8">
				<nav class="hidden items-center gap-7 lg:flex">
					<a href="#proyek" class="text-sm text-muted">Proyek</a>
					<a href="#tentang" class="text-sm text-muted">Tentang</a>
					<a href="#kontak" class="text-sm text-muted">Kontak</a>
				</nav>

				<button
					type="button"
					aria-label="Ganti tema"
					onclick={() => (theme = theme === 'dark' ? 'light' : 'dark')}
					class="flex h-11 w-11 items-center justify-center border border-line"
				>
					<svg
						width="17"
						height="17"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						stroke-width="1.5"
						stroke-linecap="round"
						stroke-linejoin="round"
						class="dark:hidden"
					>
						<path d="M21 12.8A9 9 0 1 1 11.2 3a7 7 0 0 0 9.8 9.8Z" />
					</svg>
					<svg
						width="17"
						height="17"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						stroke-width="1.5"
						stroke-linecap="round"
						class="hidden dark:block"
					>
						<circle cx="12" cy="12" r="4" />
						<path
							d="M12 2v2M12 20v2M4.9 4.9l1.4 1.4M17.7 17.7l1.4 1.4M2 12h2M20 12h2M4.9 19.1l1.4-1.4M17.7 6.3l1.4-1.4"
						/>
					</svg>
				</button>

				<button
					type="button"
					aria-label={menuOpen ? 'Tutup menu' : 'Buka menu'}
					aria-expanded={menuOpen}
					aria-controls="menu-mobile"
					onclick={() => (menuOpen = !menuOpen)}
					class="flex h-11 w-11 items-center justify-center border border-line lg:hidden"
				>
					<svg
						width="18"
						height="18"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						stroke-width="1.5"
						stroke-linecap="round"
						class={menuOpen ? 'hidden' : 'block'}
					>
						<path d="M4 7h16M4 12h16M4 17h16" />
					</svg>
					<svg
						width="18"
						height="18"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						stroke-width="1.5"
						stroke-linecap="round"
						class={menuOpen ? 'block' : 'hidden'}
					>
						<path d="M6 6l12 12M18 6L6 18" />
					</svg>
				</button>
			</div>
		</div>

		<!-- Panel menu: absolute supaya membuka menu tidak menggeser konten.
		     Kalau konten bergeser, target scroll Lenis jadi basi dan mendarat
		     beberapa puluh piksel meleset. -->
		<nav
			id="menu-mobile"
			class="absolute inset-x-0 top-full z-20 border-b border-line bg-bg lg:hidden {menuOpen
				? 'block'
				: 'hidden'}"
		>
			<div class="mx-auto flex max-w-[1440px] flex-col divide-y divide-line px-6">
				<a href="#proyek" onclick={() => (menuOpen = false)} class="py-4 text-sm text-muted">
					Proyek
				</a>
				<a href="#tentang" onclick={() => (menuOpen = false)} class="py-4 text-sm text-muted">
					Tentang
				</a>
				<a href="#kontak" onclick={() => (menuOpen = false)} class="py-4 text-sm text-muted">
					Kontak
				</a>
			</div>
		</nav>
	</header>

	<!-- hero -->
	<section class="mx-auto max-w-[1440px] px-6 pt-16 pb-14 lg:px-24 lg:pt-[120px]">
		<h1
			class="text-[62px] leading-[0.88] font-medium tracking-[-1.6px] lg:text-[232px] lg:tracking-[-9px]"
		>
			Dimas Avrian
		</h1>

		<div class="mt-7 flex flex-col gap-6">
			<p class="max-w-[560px] text-[17px] leading-[1.6] text-muted lg:text-xl">
				Siswa SMK jurusan RPL. Membangun antarmuka web yang cepat, rapi, dan mudah dipakai.
			</p>

			<div class="flex items-center gap-3">
				<a
					href="#proyek"
					class="flex h-12 items-center bg-primary px-6 text-[15px] font-medium text-on-primary"
				>
					Lihat proyek
				</a>
				<a
					href="#kontak"
					class="flex h-12 items-center border border-line px-6 text-[15px] text-ink"
				>
					Hubungi saya
				</a>
			</div>

			<p class="font-mono text-[11px] tracking-[1.1px] text-muted">
				TERSEDIA UNTUK PROYEK BARU — 2026
			</p>
		</div>
	</section>

	<!-- band -->
	<div class="border-t border-line">
		<img
			src="/images/band.jpg"
			alt="Foto sampul"
			class="h-[220px] w-full object-cover lg:h-[420px]"
		/>
		<div class="mx-auto max-w-[1440px] px-6 py-4 lg:px-24">
			<p class="font-mono text-[11px] tracking-[1.1px] text-muted">LAB RPL — SMK NEGERI 1 · 2026</p>
		</div>
	</div>

	<!-- proyek -->
	<section id="proyek" class="border-t border-line">
		<div class="mx-auto max-w-[1440px] px-6 pt-5 pb-14 lg:px-24 lg:pb-24">
			<p class="font-mono text-[11px] tracking-[1.1px]">PROYEK</p>

			<div class="mt-7 flex flex-col divide-y divide-line lg:mt-10">
				{#each projects as project, i (project.title)}
					<ProjectRow {project} index={i} />
				{/each}
			</div>
		</div>
	</section>

	<!-- tentang -->
	<section id="tentang" class="border-t border-line">
		<div class="mx-auto max-w-[1440px] px-6 pt-5 pb-14 lg:px-24 lg:pb-24">
			<p class="font-mono text-[11px] tracking-[1.1px]">TENTANG</p>

			<div class="mt-7 flex flex-col items-start gap-8 lg:mt-10 lg:flex-row lg:gap-24">
				<img
					src="/images/portrait.jpg"
					alt="Foto profil"
					class="h-[428px] w-full object-cover lg:h-[525px] lg:w-[420px]"
				/>

				<div class="flex w-full flex-col gap-7">
					<p class="max-w-[520px] text-[17px] leading-[1.6] text-muted lg:text-xl">
						{profile.bio}
					</p>

					<div class="flex flex-col divide-y divide-line">
						{#each profile.facts as fact}
							<div class="flex items-center justify-between gap-8 py-4">
								<span class="font-mono text-[11px] tracking-[1.1px] text-muted">
									{fact.label}
								</span>
								<span class="text-[15px]">{fact.value}</span>
							</div>
						{/each}
					</div>

					<div class="flex flex-col gap-3">
						<span class="font-mono text-[11px] tracking-[1.1px] text-muted">KEAHLIAN</span>
						<div class="flex flex-wrap gap-2">
							{#each profile.skills as skill}
								<span
									class="border border-line px-2.5 py-[7px] font-mono text-[11px] tracking-[1.1px] text-muted"
								>
									{skill}
								</span>
							{/each}
						</div>
					</div>
				</div>
			</div>
		</div>
	</section>

	<!-- kontak -->
	<section id="kontak" class="border-t border-line">
		<div class="mx-auto max-w-[1440px] px-6 pt-5 pb-14 lg:px-24 lg:pb-24">
			<p class="font-mono text-[11px] tracking-[1.1px]">KONTAK</p>

			<div class="mt-7 flex flex-col gap-8 lg:mt-10 lg:flex-row lg:gap-24">
				<p class="w-full text-[17px] leading-[1.6] text-muted lg:w-[420px] lg:shrink-0 lg:text-xl">
					Punya pertanyaan atau ingin bekerja sama? Kirim pesan.
				</p>

				<form
					method="POST"
					action="?/kirim"
					use:enhance={() => {
						status = 'loading';
						return async ({ result, update }) => {
							await update({ reset: result.type === 'success' });
							status = result.type === 'success' ? 'success' : 'error';
						};
					}}
					class="flex w-full flex-col gap-4 lg:flex-1"
				>
					<label class="flex flex-col gap-2">
						<span class="font-mono text-[11px] tracking-[1.1px] text-muted">NAMA</span>
						<input
							name="nama"
							required
							autocomplete="name"
							placeholder="Nama lengkap kamu"
							class="h-12 w-full border border-line bg-bg px-3.5 text-[15px] outline-none placeholder:text-muted focus:border-primary focus:ring-1 focus:ring-primary"
						/>
					</label>

					<label class="flex flex-col gap-2">
						<span class="font-mono text-[11px] tracking-[1.1px] text-muted">EMAIL</span>
						<input
							name="email"
							type="email"
							required
							autocomplete="email"
							placeholder="nama@email.com"
							class="h-12 w-full border border-line bg-bg px-3.5 text-[15px] outline-none placeholder:text-muted focus:border-primary focus:ring-1 focus:ring-primary"
						/>
					</label>

					<label class="flex flex-col gap-2">
						<span class="font-mono text-[11px] tracking-[1.1px] text-muted">PESAN</span>
						<textarea
							name="pesan"
							required
							rows="5"
							placeholder="Tulis pesan kamu di sini"
							class="min-h-[120px] w-full border border-line bg-bg px-3.5 py-3 text-[15px] outline-none placeholder:text-muted focus:border-primary focus:ring-1 focus:ring-primary"
						></textarea>
					</label>

					<button
						type="submit"
						disabled={status === 'loading'}
						class="flex h-12 items-center justify-center gap-2 border px-6 text-[15px] font-medium {status ===
						'loading'
							? 'border-line bg-surface text-muted'
							: 'border-transparent bg-primary text-on-primary'}"
					>
						{#if status === 'loading'}
							<svg
								width="16"
								height="16"
								viewBox="0 0 24 24"
								fill="none"
								stroke="currentColor"
								stroke-width="1.5"
								stroke-linecap="round"
								class="animate-spin"
							>
								<path d="M12 3a9 9 0 1 0 9 9" />
							</svg>
							Mengirim...
						{:else}
							Kirim pesan
						{/if}
					</button>

					{#if status !== 'idle'}
						<div class="flex items-center gap-3 border border-line px-4 py-3.5">
							{#if status === 'loading'}
								<svg
									width="16"
									height="16"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="1.5"
									stroke-linecap="round"
									class="animate-spin text-muted"
								>
									<path d="M12 3a9 9 0 1 0 9 9" />
								</svg>
								<span class="text-sm text-muted">Mengirim pesan...</span>
							{:else if status === 'success'}
								<svg
									width="16"
									height="16"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="1.5"
									stroke-linecap="round"
									stroke-linejoin="round"
									class="text-success"
								>
									<path d="M20 6 9 17l-5-5" />
								</svg>
								<span class="text-sm text-success">Pesan terkirim. Terima kasih!</span>
							{:else}
								<svg
									width="16"
									height="16"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="1.5"
									stroke-linecap="round"
									stroke-linejoin="round"
									class="text-danger"
								>
									<circle cx="12" cy="12" r="9" />
									<path d="M12 8v5M12 16.5v.01" />
								</svg>
								<span class="text-sm text-danger">Gagal mengirim. Coba lagi.</span>
							{/if}
						</div>
					{/if}
				</form>
			</div>
		</div>
	</section>

	<!-- footer -->
	<footer class="border-t border-line">
		<div class="mx-auto flex max-w-[1440px] flex-col gap-10 px-6 py-14 lg:px-24">
			<p
				class="max-w-[640px] text-[34px] leading-[1.05] font-medium tracking-[-0.6px] lg:text-[48px] lg:tracking-[-1.2px]"
			>
				Mari membangun sesuatu.
			</p>

			<div
				class="flex flex-col items-start gap-4 lg:flex-row lg:items-center lg:justify-between lg:gap-6"
			>
				<a
					href="mailto:dimasavrian346@gmail.com"
					class="font-mono text-xs tracking-[1.1px] text-primary"
				>
					dimasavrian346@gmail.com
				</a>
				<div class="flex items-center gap-6">
					<a href="#proyek" class="font-mono text-[11px] tracking-[1.1px] text-muted">PROYEK</a>
					<a
						href="https://github.com/FForFachriza"
						class="font-mono text-[11px] tracking-[1.1px] text-muted">GITHUB</a
					>
					<a
						href="mailto:dimasavrian346@gmail.com"
						class="font-mono text-[11px] tracking-[1.1px] text-muted"
					>
						EMAIL
					</a>
				</div>
			</div>
		</div>
	</footer>
</div>
```

## Langkah deploy

1. Push repo kalian ke GitHub.
2. Vercel: **Add New → Project**, pilih repo.
3. **Environment Variables** — tambahkan dua ini sebelum Deploy:
   - `RESEND_API_KEY`
   - `RESEND_TO`
4. Deploy. Tunggu ±1 menit.
5. Buka URL-nya. Coba dark mode. Coba kirim satu pesan.

Kalau kalian lupa langkah 3, situsnya tetap hidup — tapi form akan menjawab
"Gagal mengirim". Penyebabnya ada di **Functions → Logs**, bukan di browser.

Variabel yang ditambahkan setelah deploy pertama tidak terbaca sampai kalian
menekan **Deployments → ⋯ → Redeploy**. `.env` tidak ikut terkirim ke Vercel,
dan itu memang tujuannya.

## Tiga keputusan di sentuhan akhir

**Panel menu sengaja `absolute`, bukan menggeser konten.** Kalau panel membuka
dan mendorong hero ke bawah, Lenis sudah menghitung posisi tujuan sebelum
dorongan itu terjadi — hasilnya mendarat beberapa puluh piksel meleset. Panel
yang menumpuk di atas konten menghilangkan seluruh masalah itu.

**Ikon hamburger dan X ditukar dengan kelas, bukan `{#if}`.** Bukan karena
`{#if}` salah — di CP04 justru itu yang dipakai. Di sini kelas lebih murah,
karena kedua ikon toh sudah ada di DOM.

**Lenis dimatikan kalau sistem minta gerak diminimalkan.** Satu baris
`matchMedia('(prefers-reduced-motion: reduce)')`. Scroll yang dipaksa halus
bisa bikin pusing; menghormati setelan sistem bukan fitur tambahan, itu syarat.

## Bandingkan

/compare/06-resend...07-deploy

## Tantangan

Buka URL kalian dari HP teman. Tiga hal yang paling sering rusak di layar
kecil: nama yang meluber, gambar yang tidak punya `object-cover`, dan tombol
yang terlalu kecil untuk jempol.

Buka menu di HP, lalu tekan `Escape` — panel harus tertutup. Sekarang coba
buka menu dan klik di luar panel: apa yang terjadi? Kalau kalian mau panelnya
ikut tertutup, apa yang perlu ditambahkan?

Bonus: ganti `lerp` Lenis jadi `0.05`, lalu `0.15`. Mana yang terasa lebih
"berat"? Tidak ada jawaban benar, tapi sekarang kalian bisa memilih.

## Kalau macet

| Gejala                                           | Sebab                                                                                         |
| ------------------------------------------------ | --------------------------------------------------------------------------------------------- |
| Build gagal: `adapter` tidak dikenal             | `bun add -d @sveltejs/adapter-vercel` belum dijalankan, atau `bun.lock` tidak ikut ter-commit |
| Situs 404 di semua halaman                       | Framework Preset di Vercel bukan SvelteKit                                                    |
| Form selalu gagal di Vercel, padahal lokal jalan | Environment Variables belum diisi, atau belum Redeploy                                        |
| Gambar hilang                                    | File ada di `static/images/` tapi tidak ikut ter-commit                                       |
| Email terkirim, tapi ke alamat aneh              | `RESEND_TO` di Vercel masih nilai contoh                                                      |
| Link `#proyek` tidak menggulir                   | `new Lenis()` tanpa `anchors: true` — Lenis memblokir anchor secara default                   |
| Scroll tersendat / terasa dua kali               | Ada `scroll-behavior: smooth` di CSS. Hapus; jangan dicampur Lenis                            |
| Menu terbuka, konten di bawahnya bergeser        | Panel tidak `absolute`, atau header lupa `relative`                                           |

## Verifikasi

```sh
bun run check
bun run dev
```

Di 390px: tombol menu membuka panel berisi tiga link, klik salah satunya
menutup panel dan menggulir ke section yang dituju, `Escape` juga menutup.
Di 1440px: form selebar sisa ruang, tombol kirim selebar form, `html` punya
kelas `lenis`, dan tidak ada scroll ke samping.

Lalu buka URL Vercel kalian di jendela penyamaran (bukan yang sudah login).
Semua harus jalan tanpa kalian menyentuh terminal:

- nama besar di atas, foto band di tengah
- tombol tema mengubah seluruh halaman
- tiga proyek, section Tentang, form kontak
- satu pesan sungguhan masuk ke inbox

Kalau semuanya lolos, portofolio kalian sudah online.
