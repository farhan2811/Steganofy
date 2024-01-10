<script>
	import { onMount } from 'svelte';


	onMount(() => {
		let demo = document.getElementById('title');
		let typewriter = new Typewriter(demo, {
		    strings: ['Steganofy'],
		    delay: 'natural', // 'natural' or Number
		    loop: true, // infinite loop
		    autoStart: true,
		});
	})
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Home Steganofy" />
</svelte:head>

<div class="landing flex flex-direction-col flex-center-vertical flex-center-horizontal flex-gap-large">
	<div class="title-landing" id="title"></div>
	<div class="sub-title-landing">Conceal & Reveal. Keep Your Secrets "Confidential".</div>
	<a href="/steganography" class="button">Start Your Privacy Journey Here</a>
</div>
<div class="footer flex flex-center-horizontal flex-center-vertical">Made by Farhan & Riandy. Starts With Bismillah & Ends In Alhamdulillah.</div>
