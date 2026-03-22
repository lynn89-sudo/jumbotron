<script>
    import { base } from "$app/paths";
    import { page } from "$app/state";
    //import { onMount } from "svelte";

    import { sync } from "$lib/sync.svelte.js";
    import { tutorial } from "$lib/sync.svelte.js";
    import { onMount } from "svelte";

    let announcementTitle = $state("");
    let announcementMessage = $state("");
    let eventsTitle = $state("");
    let eventsTime = $state();

    let time = new Date();
    let placeholderTime = $state("13:30"); 

    let timerID = "";

    let formatLabel = $state("AM/PM")
    function switchFormat() {
        if (formatLabel == "AM/PM") {
            formatLabel = "International";
        }
        else {
            formatLabel = "AM/PM";
        }
    }

    onMount(function() {
        if (localStorage.getItem("jumbotron.announcement.title") == null || localStorage.getItem("jumbotron.announcement.message") == null) {
            localStorage.setItem("jumbotron.announcement.title", "");
            localStorage.setItem("jumbotron.announcement.message", "");
        }
        if (localStorage.getItem("jumbotron.event.title") == null || localStorage.getItem("jumbotron.event.time") == null) {
            localStorage.setItem("jumbotron.event.title", "");
            localStorage.setItem("jumbotron.event.time", "");
            localStorage.setItem("jumbotron.event.label", "");
        }
    })

    

    //console.log(time.getHours());
    /*
    onMount(function() {
        
        placeholderTime = "";
        if (time.getHours() == 0) {
            placeholderTime += "12:";
            if (time.getMinutes() <= 9) {
                placeholderTime += "0";
            }
            placeholderTime += time.getMinutes();
            placeholderTime += " AM";
        }
        else if (time.getHours() > 11) {
            placeholderTime += (time.getHours() - 12);
            placeholderTime += ":";
            if (time.getMinutes() <= 9) {
                placeholderTime += "0";
            }
            placeholderTime += time.getMinutes();
            placeholderTime += " PM";
        }
        else {
            placeholderTime += (time.getHours());
            placeholderTime += ":";
            if (time.getMinutes() <= 9) {
                placeholderTime += "0";
            }
            placeholderTime += time.getMinutes();
            placeholderTime += " AM";
        }
            
    });

    */

    function formatTimeLabel(raw) {
        if (formatLabel == "AM/PM") {
            return raw;
        }
        let splitted = raw.split(":");
        if (splitted[0] > 12) {
            let rtn = splitted[0]-12;
            rtn += ":";
            rtn += splitted[1];
            rtn += " PM";
            return rtn;
        }
        else if (splitted[0] == 12) {
            let rtn = "12";
            rtn += ":";
            rtn += splitted[1];
            rtn += " PM";
            return rtn;
        }
        else if (splitted[0] == 0) {
            let rtn = "12";
            rtn += ":";
            rtn += splitted[1];
            rtn += " AM";
            return rtn;
        }
        else {
            let rtn = splitted[0];
            rtn += ":";
            rtn += splitted[1];
            rtn += " AM";
            return rtn;
        }

    }

    function syncAnnouncements() {
        sync.announcements = true;
        localStorage.setItem("jumbotron.sync", false);
        clearInterval(timerID);
        //console.log("Syncing:", { announcementTitle, announcementMessage, eventsTitle });
        localStorage.setItem("jumbotron.announcement.title", announcementTitle);
        localStorage.setItem("jumbotron.announcement.message", announcementMessage);
        localStorage.setItem("jumbotron.event.title", eventsTitle);
        localStorage.setItem("jumbotron.event.time", eventsTime);
        localStorage.setItem("jumbotron.event.label", formatTimeLabel(eventsTime));
        setTimeout(function() {localStorage.setItem("jumbotron.sync", true)}, 2000);
        setTimeout(function() {sync.announcements = false; localStorage.setItem("jumbotron.sync", false)}, 3000);
        setTimeout(setAlarm, 3000);
    }

    function setAlarm() {
        if (eventsTitle == "" || eventsTime == "") {
            return;
        }
        let split = eventsTime.split(":");
        let time = new Date();
        if (time.getHours() > split[0]) {
            window.alert("If this event was intended to occur today, the time of this event has already passed. If this event is intended to occur tomorrow, you will need to sync again tomorrow in order for live updates to occur.");
            return
        }
        else if (time.getMinutes() > split[1] && time.getHours() == split[0]) {
            window.alert("If this event was intended to occur today, the time of this event has already passed. If this event is intended to occur tomorrow, you will need to sync again tomorrow in order for live updates to occur.");
            return
        }
        let t1 = (split[0]*60 + parseInt(split[1])); // Event
        let t2 = (time.getHours()*60) + time.getMinutes(); // Current Time
        timerID = setInterval(() => {
            time = new Date();
            t2 = (time.getHours()*60) + time.getMinutes();
            if (t1-t2 <= 0) {
                localStorage.setItem("jumbotron.event.label", "Now");
                localStorage.setItem("jumbotron.sync", true);
                setTimeout(() => {localStorage.setItem("jumbotron.sync", false); clearInterval(timerID);}, 1000);
            }
            else if (t1-t2 == 1) {
                localStorage.setItem("jumbotron.event.label", "In 1 minute");
                localStorage.setItem("jumbotron.sync", true);
                setTimeout(() => {localStorage.setItem("jumbotron.sync", false)}, 1000);
            }
            else if (t1-t2 <= 30) {
                localStorage.setItem("jumbotron.event.label", "In " + (t1-t2) + " minutes");
                localStorage.setItem("jumbotron.sync", true);
                setTimeout(() => {localStorage.setItem("jumbotron.sync", false)}, 1000);
            }
            //console.log(timerID);
        }, 2000)
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
{#if tutorial.enabled}<p>You can modify the announcement popup by filling out the form fields below. The popup will appear when you sync the display window while the form fields contain content, and the popup will dissapear when you sync the display window and the form fields contain no content.</p>{/if}
<form>
    <input bind:value={announcementTitle} type = "text" placeholder="Title"><br>
    <input bind:value={announcementMessage} class="bigInput" type="text" placeholder="Message">
</form>
<h4 style:margin-top=10px>Scheduled Event</h4>
{#if tutorial.enabled}<p>You can modify the upcoming event module by filling out the form fields below. The module will appear when you sync the display window while the form fields contain content, and will count down the time until your event. If the time has already passed, your event will be labelled as currently happening. The module will dissapear when you sync the display window and the form fields contain no content.</p>{/if}
<form>
    <input bind:value={eventsTitle} type="text" placeholder="Title">
    <input bind:value={eventsTime} type="time" placeholder={placeholderTime}>
    {#if formatLabel == "International"}
    <button onclick={switchFormat}>Displaying AM/PM Format</button>
    {:else}
    <button onclick={switchFormat}>Displaying International Format</button>
    {/if}
</form>
{#if tutorial.enabled}<p>International Format will display the time of your event in the 24 hour clock; AM/PM format will display the time of your event in the 12 hour clock. Clicking the button above toggles between formats.</p>{/if}
<!--{#if tutorial.enabled}<p>Use the 24hr clock to configure the time of your event.</p>{/if}-->
<p><button class:disabled={sync.announcements} onclick={syncAnnouncements} disabled={sync.announcements}>Sync Announcements and Events on Display Windows</button></p>