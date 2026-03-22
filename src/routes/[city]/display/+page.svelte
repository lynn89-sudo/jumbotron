<script>
    import { base } from "$app/paths";
    import { page } from "$app/state";
    import { onMount } from "svelte";
    import { fade, fly, slide } from "svelte/transition"

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
    let announcement = $state([]);

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
            
            if (localStorage.getItem("jumbotron.announcement.title") == "" || localStorage.getItem("jumbotron.announcement.message") == "") {
                announcement = [];
                console.log("Cleared announcement");
            }
            else {
                announcement = [localStorage.getItem("jumbotron.announcement.title"), localStorage.getItem("jumbotron.announcement.message")]
                console.log("Synced announcement");
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
        top: 55px;

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
    #event.now {
        animation: eventPulse 1s ease-in-out infinite;
    }
    @keyframes eventPulse {
        0%, 100% {
            box-shadow: 0px 0px 10px 5px rgba(0, 0, 0, 0);
        }
        50% {
            box-shadow: 0px 0px 10px 5px rgba(0, 0, 0, 0.205);
        }
    }

    #announcement {
        position: fixed;
        transform: translate(-50%, -50%);
        left: 50%;
        top: 50%;
        background-color: grey;
        color: white;
        width: 90%;
        padding: 20px;
        border-radius: 25px;
        z-index: 1000;

        div {
            padding: 5px;
            margin: 10px;
            margin-top: 20px;
            border-radius: 20px;
            background-color: rgb(102, 102, 102);
            p {
                color: white;
                font-size: 20px;
            }
        }
    }
    #announcement-smoke {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(44, 44, 44, 0.863);
    }
</style>

<svelte:head>
    <title>{proccessEventCity(page.params.city)}</title>
</svelte:head>
{#if event.length > 0}
<div id="event" class:now={event[1] == "Now"} transition:slide>
    <p><strong>{event[0]}</strong> <span id="timeLabel">{event[1]}</span></p>
</div>
{/if}
{#if announcement.length > 0}
<div id="announcement-smoke" transition:fade={{delay: 1000}}></div>
<div id="announcement" transition:fly={{y:200}}>
    <h1>{announcement[0]}</h1>
    <!--<span class="material-symbols-outlined">circle_notifications</span>-->
    <div>
        <p>{announcement[1]}</p>
    </div>
</div>
{/if}