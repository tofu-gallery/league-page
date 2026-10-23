<!-- __layout.svelte -->
<script>
	import { Nav, Footer } from "$lib/components"
    // Tofu gallery: upstream's Vercel Web Analytics (injectAnalytics) is removed so copies carry no tracking. See TOFU.md.
</script>

<main>
    <Nav /> <!-- adds the nav (small and large) -->
  
    <slot />

    <Footer /> <!-- adds the footer -->
</main>