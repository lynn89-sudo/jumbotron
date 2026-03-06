<script>

    import { base } from "$app/paths";
    import { page } from "$app/state";
    import { onMount } from "svelte";

    import { proccessEventCity } from "$lib/event.js";
    import { proccessCity } from "$lib/event.js";
    import { checkCity } from "$lib/event.js";

    import MainConfig from "$lib/configs/main.svelte";
    import AnnouncementConfig from "$lib/configs/announcements.svelte";
    import { sync } from "$lib/sync.svelte";

    onMount(async function() {
        console.log("Checking city name...")
       if (await checkCity(proccessCity(page.params.city)) == true) {
        console.log("City name is valid!")
       }
       else {
        window.location.href = base + "/" // this should eventually link to an error page
       }
    })
    /*
    List of features
    - Set the next event and its time
    - Announcements popup
    - Set a timer
    - Mode to view a google slides
    */
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
        bottom: -150px;
        left: 50%;
        transition: all 0.3s ease-in-out;
        background-color: rgb(177, 82, 82);
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
</style>
<div id="syncingNotif" class:active={sync.enabled || sync.announcements}>
    <p>Your display windows are syncing. Hang tight!</p>
</div>
<h1>{proccessEventCity(page.params.city)}</h1>
<h2 style:font-size=20px style:font-weight=400 style:margin-bottom=30px>Jumbotron Control Panel</h2>
<p>You should keep this window open on your laptop or main screen, and open the display window on another screen.</p>
<p style:margin-bottom=40px><button onclick={function() {window.location.href=base+"/" + page.params.city + "/control/guide"}}>How to use Jumbotron</button></p>
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
    <p>Seamlessly display a Google Slides or a PDF to your audience.</p>
</div>
<div id="timer" class="config">
    <h3>Timers</h3>
    <p>Set a timer and project it onto your display.</p>
</div>