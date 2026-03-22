<script>
    import { base } from "$app/paths";
    import { page } from "$app/state";
    import { onMount } from "svelte";
    import { slide } from "svelte/transition"

    import { proccessEventCity } from "$lib/event.js";
    import { proccessCity } from "$lib/event.js";
    import { checkCity } from "$lib/event.js";

    onMount(async function() {
        console.log("Checking city name...")
       if (await checkCity(proccessCity(page.params.city)) == true) {
        console.log("City name is valid!")
       }
       else {
        window.location.href = base + "/" // this should eventually link to an error page
       }
    })

    let event = $state([]);

    onMount(() => {
        window.addEventListener("storage", (e) => {
            sync();
        })
        setTimeout(sync, 1000);
    })

    function sync() {
        if (localStorage.getItem("jumbotron.sync") == "true") {
            console.log("Started display sync");
            if (localStorage.getItem("jumbotron.event.title") == "" || localStorage.getItem("jumbotron.event.time") == "undefined" || localStorage.getItem("jumbotron.event.time") == "") {
                event = [];
                console.log("Cleared event");
            }
            else {
                event = [localStorage.getItem("jumbotron.event.title"), localStorage.getItem("jumbotron.event.label")]
                console.log("Synced event");
            }
        }
        console.log("Finished display sync");
        //console.log(event);
    }

</script>
<style>
    #event {
        padding: 5px;
        background-color: rgba(94, 93, 93, 0.671);
        border-radius: 35px;
        position: fixed;
        transform: translate(-50%, -50%);
        left: 50%;
        width: 40%;
        top: -200px;

        #timeLabel {
            border-radius: 30px;
            background-color: grey;
            margin-left: 10px;
            padding: 10px;
        }
        p {
            color: white;
        }

    }
    #event.active {
        top: 50px;
    }
</style>

<svelte:head>
    <title>{proccessEventCity(page.params.city)}</title>
</svelte:head>
{#if event.length > 0}
<div id="event" class:active={event != []} transition:slide>
    <p><strong>{event[0]}</strong> <span id="timeLabel">{event[1]}</span></p>
</div>
{/if}