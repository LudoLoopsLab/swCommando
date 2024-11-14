<script lang="ts">
	import '../app.css';
	import { Navbar, NavBrand, NavLi, NavUl, NavHamburger } from 'flowbite-svelte';
	let { children } = $props();
</script>

<Navbar>
	<NavHamburger />
	<NavUl>
		<NavLi href="/">Clone Trooper</NavLi>
		<NavLi href="/deathtrooper">Death Trooper</NavLi>
	</NavUl>
</Navbar>

{@render children()}
