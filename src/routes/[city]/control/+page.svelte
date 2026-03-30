<script>

    import { base } from "$app/paths";
    import { page } from "$app/state";
    import { onMount } from "svelte";

    import { proccessEventCity } from "$lib/event.js";
    import { proccessCity } from "$lib/event.js";
    import { checkCity } from "$lib/event.js";

    import MainConfig from "$lib/configs/main.svelte";
    import AnnouncementConfig from "$lib/configs/announcements.svelte";
    import Slides from "$lib/configs/slides.svelte";
    import Liveshare from "$lib/configs/liveshare.svelte";
    import { sync, tutorial } from "$lib/sync.svelte";

    onMount(async function() {
        console.log("Checking city name...")
       if (await checkCity(proccessCity(page.params.city)) == true) {
        console.log("City name is valid!")
       }
       else {
        window.location.href = base + "/" // this should eventually link to an error page
       }
    })

    function toggleTut() {
        if (tutorial.enabled) {
            tutorial.enabled = false;
        }
        else {
            tutorial.enabled = true;
        }
    }

    function forceSync() {
        localStorage.setItem("jumbotron.sync", true);
        sync.enabled = true;
        setTimeout(() => {localStorage.setItem("jumbotron.sync", false); sync.enabled = false;}, 2000)
    }
    

    let wakeLock = null;

    // Replace your existing requestWakeLock and onMount logic with this:

    async function requestWakeLock() {
        if ('wakeLock' in navigator) {
            try {
                wakeLock = await navigator.wakeLock.request('screen');
                console.log("WakeLock enabled");
                
                wakeLock.addEventListener("release", () => {
                    console.log("WakeLock was released");
                });
            } catch (err) {
                console.error(`${err.name}, ${err.message}`);
            }
        }
    }

    onMount(() => {
        requestWakeLock();

        // Re-request lock if the tab was hidden and comes back to focus
        const handleVisibilityChange = async () => {
            if (document.visibilityState === 'visible') {
                await requestWakeLock();
            }
        };

        document.addEventListener('visibilitychange', handleVisibilityChange);
        return () => document.removeEventListener('visibilitychange', handleVisibilityChange);
    });

    onMount(() => {requestWakeLock();})
</script>

<svelte:head>
    <title>Control Window | Jumbotron</title>
</svelte:head>
<style>
    .config {
        margin: 20px;
        background-color: rgb(231, 231, 231);
        padding: 20px;
        border-radius: 30px;
        border: 5px grey solid;
        h3 {
            margin-bottom: 10px;
        }
        
    }
    #syncingNotif {
        position: fixed;
        z-index: 1000;
        transform: translate(-50%, -50%);
        bottom: -550px;
        left: 50%;
        transition: all 0.7s ease-in-out;
        background-color: rgba(177, 82, 82, 0.948);
        box-shadow: 0px 0px 20px 10px rgba(0, 0, 0, 0.237);
        border-radius: 40px;
        padding: 20px;
        p {
            color: white;
        }
    }
    #syncingNotif.active {
        bottom: 10px;
    }

    #sync {
        position: fixed;
        top: 10px;
        right: 25px;
    }

    .spin {
        animation: spin 0.6s ease-in-out infinite;
    }
    @keyframes spin {
        0% {
            transform: rotate(0deg)
        }
        50% {
            transform: rotate(180deg)
        }
        100% {
            transform: rotate(360deg)
        }
    }
</style>
<div id="syncingNotif" class:active={sync.enabled || sync.announcements || sync.slides}>
    <p>Your display windows are syncing. Hang tight!</p>
</div>
<h1>{proccessEventCity(page.params.city)}</h1>
<h2 style:font-size=20px style:font-weight=400 style:margin-bottom=30px>Jumbotron Control Panel</h2>
<p>You should keep this window open on your laptop or main screen, and open the display window on another screen.</p>
<p style:margin-bottom=40px><button onclick={toggleTut}>{#if tutorial.enabled}Hide Tutorial{:else}Show Tutorial{/if}</button></p>
<div id="main" class="config">
    <h3>Main Configuration</h3>
    <MainConfig />
</div>
<div id="events" class="config">
    <h3>Announcements and Events</h3>
    <AnnouncementConfig title=""/>
</div>
<div id="presentations" class="config">
    <h3>Presentations</h3>
    <Slides />
</div>
<div id="liveshare" class="config">
    <h3>Liveshare</h3>
    <Liveshare />
</div>
{#if false}
<button id="sync" title="Sync Displays" onclick={forceSync}>
    <span class="material-symbols-outlined" class:spin={sync.enabled || sync.announcements || sync.slides}>sync</span>
</button>
{/if}