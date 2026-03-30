<script>
    import { base } from "$app/paths";
    import { page } from "$app/state";
    import { onMount } from "svelte";
    import { blur, fade, fly, slide } from "svelte/transition"

    import { eventName, proccessEventCity } from "$lib/event.js";
    import { proccessCity } from "$lib/event.js";
    import { checkCity } from "$lib/event.js";

    let screenY = $state(0);
    let screenX = $state(0);

    onMount(async function() {
        console.log("Checking city name...")
       if (await checkCity(proccessCity(page.params.city)) == true) {
        console.log("City name is valid!")
       }
       else {
        window.location.href = base + "/" // this should eventually link to an error page
       }
    })

    onMount(() => {
        window.addEventListener("keypress", (e) => {
            if (e.key == "f") {
                document.documentElement.requestFullscreen();
                requestWakeLock();
            }
        })
    })

    let event = $state([]);
    let announcement = $state([]);
    let presentation = $state("");

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

            if (localStorage.getItem("jumbotron.googleLink") == "" && localStorage.getItem("jumbotron.fileLink") == "") {
                presentation = "";
                console.log("Cleared presentation");
            }
            else if (localStorage.getItem("jumbotron.googleLink") != "") {
                presentation = localStorage.getItem("jumbotron.googleLink");
                //setTimeout(() => {document.getElementById("presentation").requestFullscreen();}, 100)
            }
            else {
                presentation = localStorage.getItem("jumbotron.fileLink");
            }
        }
        console.log("Finished display sync");
        //console.log(event);
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
        background-color: rgba(44, 44, 44, 0.971);
        z-index: 999
    }

    #presentation {
        position: fixed;
        transform: translate(-50%, -50%);
        left: 50%;
        top: 50%;
        z-index: 998
    }

    #eventNameDisplay {
        position: fixed;
        left: 20px;
        bottom: 20px;
        h1 {
            color: rgb(85, 40, 14);
            font-size: 90px;
            text-align: left;

            span {
                color: rgb(188, 103, 54);
            }
        }
    }
</style>
<svelte:window bind:innerHeight={screenY} bind:innerWidth={screenX}></svelte:window>
<svelte:head>
    <title>{proccessEventCity(page.params.city)}</title>
</svelte:head>
{#if event.length > 0}
<div id="event" class:now={event[1] == "Now"} transition:slide>
    <p><strong>{event[0]}</strong> <span id="timeLabel">{event[1]}</span></p>
</div>
{/if}
{#if announcement.length > 0}
<div id="announcement-smoke" in:fade out:fade={{delay: 1000}}></div>
<div id="announcement" in:fly={{y:400, duration:1500, delay: 1000}} out:fly={{y:400, duration:1500}}>
    <h1>{announcement[0]}</h1>
    <!--<span class="material-symbols-outlined">circle_notifications</span>-->
    <div>
        <p>{announcement[1]}</p>
    </div>
</div>
{/if}
{#if presentation != ""}
<iframe transition:blur id="presentation" src={presentation} width={screenX-10} height={screenY-10} title="Presentation"></iframe>
{/if}
<div id="eventNameDisplay">
    <h1><span>{eventName}</span><br>{proccessCity(page.params.city)}</h1>
</div>