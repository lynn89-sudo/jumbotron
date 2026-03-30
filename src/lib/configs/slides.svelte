<script>
    import { base } from "$app/paths";
    import { page } from "$app/state";
    import { onMount, unmount } from "svelte";
    import { slide } from "svelte/transition";

    import { sync } from "$lib/sync.svelte.js";
    import { tutorial } from "$lib/sync.svelte.js";

    //let googleLink = $state("");
    let fileLink = $state("");
    let mountedEnabled = $state(false);

    onMount(function() {
        //localStorage.setItem("jumbotron.googleLink", "");
        localStorage.setItem("jumbotron.fileLink", "");
    })

    //https://docs.google.com/viewer?url=YOUR_PUBLIC_FILE_URL&embedded=true

    /*
    $effect(function() {
        if (sync.enabled == true) {

            localStorage.setItem("jumbotron.announcement.title", announcementTitle);
            localStorage.setItem("jumbotron.announcement.message", announcementMessage);
            localStorage.setItem("jumbotron.event.title", eventsTitle);
            localStorage.setItem("jumbotron.event.time", eventsTime);

        }
    })
    */

    /*
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
        */

        /*
    function enableGoogle() {
        sync.slides = true;
        document.getElementById("google").disabled = true;
        if (googleLink.indexOf("<iframe") != -1) {
            let str = googleLink.split(`"`);
            localStorage.setItem("jumbotron.googleLink", str[1]);
        }
        else {
            localStorage.setItem("jumbotron.googleLink", googleLink);
        }
        localStorage.setItem("jumbotron.fileLink", "");
        setTimeout(function() {localStorage.setItem("jumbotron.sync", true); sync.liveshare = true;}, 2000);
        setTimeout(function() {sync.slides = false; localStorage.setItem("jumbotron.sync", false); sync.liveshare = false; document.getElementById("google").disabled = false; mountedEnabled = true;}, 3000)
        setTimeout(() => {checkLink(localStorage.getItem("jumbotron.googleLink"), "google")}, 3500);
    }
        */

    function enableFile() {
        sync.slides = true;
        document.getElementById("file").disabled = true;
        localStorage.setItem("jumbotron.googleLink", "");
        let split = fileLink.split("/");
        let link = `https://drive.google.com/file/d/${split[5]}/preview`
        localStorage.setItem("jumbotron.fileLink", link);
        setTimeout(function() {localStorage.setItem("jumbotron.sync", true); sync.liveshare = true;}, 2000);
        setTimeout(function() {sync.slides = false; localStorage.setItem("jumbotron.sync", false); sync.liveshare = false; document.getElementById("file").disabled = false; mountedEnabled = true;}, 3000)
        setTimeout(() => {checkLink(fileLink, "file")}, 3500);
    }

    function unmountDisplay() {
        sync.slides = true;
        //localStorage.setItem("jumbotron.googleLink", "");
        localStorage.setItem("jumbotron.fileLink", "");
        mountedEnabled = false;
        setTimeout(function() {localStorage.setItem("jumbotron.sync", true); sync.liveshare = true;}, 2000);
        setTimeout(function() {localStorage.setItem("jumbotron.sync", false); sync.slides = false; sync.liveshare = false;}, 3000)
    }

    async function checkLink(link, type) {
        return true;
        /*
        console.log(type)
        if (type == "google") {
            console.log("Checking google link");
            if (link.indexOf("https://docs.google.com/presentation") == -1 || link.indexOf("pubembed") == -1) {
                unmountDisplay();
                window.alert("The format of the provided link does not match with Google Slides. Link will be unmounted from displays.");
                return;
            }
        }
        else if (type == "file") {
            console.log("Checking file link"); //https://drive.google.com/file/d/16AS73qqHN1bS1YzrjH34Fel4h7o2tuIV/view?usp=share_link
            if (link.indexOf("https://drive.google.com/file") == -1 || link.indexOf("view?usp=share_link") == -1) {
                unmountDisplay();
                window.alert("The format of the provided link does not match with Google Drive. Link will be unmounted from displays.");
                return;
            }
        }
            */
        let check = await fetch(link);
        //console.log(check.ok)
        if (!check.ok) {
            unmountDisplay();
            window.alert("Provided link does not work. Link will be unmounted from displays.");
            return;
        }
    }

    let savedLinks = [];
    function modifyLinks(mode, link) {
        if (mode) {
            savedLinks.push(link);
        }
        else {
            savedLinks = savedLinks.filter((item) => item != link);
        }
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
    h4 {
        margin-top: 35px;
    }
</style>
{#if !mountedEnabled}
<div transition:slide>
<!--
    <h4>Google Slides</h4>
    {#if tutorial.enabled}<p>To display a Google Slides Presentation on your display windows, go to your Google Slides Presentation, find <i>Publish to Web</i>, choose Embed, copy and then paste the provided link below. You can also copy the entire embed given by Google.</p>{/if}
    <form> 
        <input bind:value={googleLink} class="bigInput" type="url" placeholder="https://docs.google.com/presentation...">
    </form>
    <p><button onclick={enableGoogle} id="google" class:disabled={sync.slides}>Display Google Slides on Display Windows</button></p>
    {#if tutorial.enabled}<p>Note that your progression through the slides are individual to the display window. Multiple display windows will not progress through the slides together.</p>{/if}
-->
    <h4>Google Drive Files</h4>
    {#if tutorial.enabled}<p>To display a file on your display window, you will need to find your file in Google Drive. From there, find share and copy the link. Paste that link below. This will also work for Google Slides.</p>{/if}
    <p>You should confirm that your document is publically accessible.</p>
    <form> 
        <input bind:value={fileLink} class="bigInput" type="url" placeholder="https://drive.google.com/file...">
    </form>
    {#if tutorial.enabled}<p>Note that your progression through the document is individual to the display window, and you will need to use scroll or arrow keys to navigate the document. Multiple display windows will not progress through the document together.</p>{/if}
    <p><button onclick={enableFile} id="file" class:disabled={sync.slides}>Display File on Display Windows</button></p>
</div>
{:else}
<div transition:slide>
    <p>Currently displaying <strong>{fileLink}</strong></p>
    <p>Your content should now be mounted on your display windows. Use the button below to unmount your content. Content cannot be changed until unmounted.</p>
    {#if sync.liveshare}<p>Note that participants who have connection to your liveshare have unrestricted access to your file/presentation; pages and slides are not limited to your current progression through the document. It is additionally imperative that your public access link does not grant editing permissions.</p>{/if}
    <p><button onclick={unmountDisplay}>Unmount Content from Display Windows</button></p>
</div>
{/if}