<script>
    import { base } from "$app/paths";
    import { page } from "$app/state";
    import { onMount } from "svelte";

    import { sync } from "$lib/sync.svelte.js";

    onMount(function() {
       localStorage.setItem("jumbotron.sync", false);
    })

    function openDisplay() {
        window.open(base + "/" + page.params.city + "/display")
    }
    function syncDisplay() {
        sync.enabled = true;
        document.getElementById("sync").disabled = true;
        setTimeout(function() {localStorage.setItem("jumbotron.sync", true)}, 2000)
        setTimeout(function() {sync.enabled = false; document.getElementById("sync").disabled = false; localStorage.setItem("jumbotron.sync", false)}, 3000)
    }
</script>
<style>
    button {
        background-color: rgb(92, 89, 89);
    }
    button.disabled {
        cursor: progress;
    }
</style>
<h4>Display Windows</h4>
<p>Display windows should be moved to another screen. Once you are done making changes from this window, you can use the button below to sync all changes.</p>
<p><button onclick={openDisplay}>Open New Display Window</button><button onclick={syncDisplay} id="sync" class:disabled={sync.enabled == true}>Sync All Changes to Display Windows</button></p>
<p>New windows are not automatically synced. Sync the windows using the button above when you are ready.</p>
