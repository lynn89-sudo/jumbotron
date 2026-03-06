<script>
    import { base } from "$app/paths";
    import { page } from "$app/state";
    //import { onMount } from "svelte";

    import { sync } from "$lib/sync.svelte.js";

    let announcementTitle = $state("");
    let announcementMessage = $state("");
    let eventsTitle = $state("");
    let eventsTime = $state();

    $effect(function() {
        if (sync.enabled == true) {
            localStorage.setItem("jumbotron.announcement.title", announcementTitle);
            localStorage.setItem("jumbotron.announcement.message", announcementMessage);
            localStorage.setItem("jumbotron.event.title", eventsTitle);
            localStorage.setItem("jumbotron.event.time", eventsTime);
        }
    })

    function syncAnnouncements() {
        sync.announcements = true;
        document.getElementById("sync").disabled = true;
        localStorage.setItem("jumbotron.announcement.title", announcementTitle);
        localStorage.setItem("jumbotron.announcement.message", announcementMessage);
        localStorage.setItem("jumbotron.event.title", eventsTitle);
        localStorage.setItem("jumbotron.event.time", eventsTime);
        setTimeout(function() {localStorage.setItem("jumbotron.sync", true)}, 2000);
        setTimeout(function() {sync.announcements = false; localStorage.setItem("jumbotron.sync", false); document.getElementById("sync").disabled = false}, 3000)
    }
    
</script>

<style>
    form {
        input {
            margin: 8px;
            border-radius: 15px;
            padding: 10px; 
        }
    }
    button {
        background-color: rgb(92, 89, 89);
    }
    button.disabled {
        cursor: progress;
    }
</style>
<h4>Announcements</h4>
<p>You can modify the announcement popup by filling out the form fields below. The popup will appear when you sync the display window while the form fields contain content, and the popup will dissapear when you sync the display window and the form fields contain no content.</p>
<form>
    <input bind:value={announcementTitle} placeholder="Title"><br>
    <input bind:value={announcementMessage} class="bigInput" type="text" placeholder="Message">
</form>
<h4 style:margin-top=10px>Scheduled Events</h4>
<p>You can modify the upcoming events module by filling out the form fields below. The module will appear when you sync the display window while the form fields contain content, and will count down the time until your event. If the time has already passed, your event will be labelled as currently happening. The module will dissapear when you sync the display window and the form fields contain no content.</p>
<form>
    <input bind:value={eventsTitle} placeholder="Title">
    <input bind:value={eventsTime} type="time">
</form>
<p>Use the 24hr clock to configure the time of your event.</p>
<p><button class:disabled={sync.announcements} onclick={syncAnnouncements} id="sync">Sync Announcements and Events on Display Windows</button></p>