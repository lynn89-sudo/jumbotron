<script>
    import { page } from "$app/state";
    import { base } from "$app/paths";
    import { onMount } from "svelte";
    import { eventName } from "$lib/event.js";
    import Peer from "peerjs";
    import { fly, slide } from "svelte/transition";

    let ip = $state("");

    onMount(async () => {
        if (page.params.city == "live") {
            let ipRaw = await fetch("https://api.ipify.org/?format=json");
            let ipJson = await ipRaw.json();
            ip = ipJson.ip;
        }
        else {
            window.location.href = base + "/" // Eventually should route to an error page
        }
    })

    let mode = $state(0);
    let peer;
    let dataConnection;

    let pong = "";

    let packet = $state({});

    let inputtedId = $state("");


    function connectToEvent() {
        //event.preventDefault();
        console.log("Attempting connection");
        peer = new Peer();
        peer.on("open", () => {
            dataConnection = peer.connect(`hackclub${eventName}${inputtedId}`);
            dataConnection.on("error", (err) => {
                console.log(err.type);
                if (err.type == "not-open-yet") {
                    //clearInterval(pong);
                    console.log("Peer not found");
                }
            })
            dataConnection.on("open", () => {
                console.log("Connection established");
                dataConnection.send({
                    "ip": ip
                })
                mode = 1;
            })
            dataConnection.on("close", () => {
                //clearInterval(pong);
                console.log("Connection closed");
                window.alert("The connection to your event's Jumbotron has been lost. If you think this was a mistake, please ask for help.")
                mode = 0;
            })
            dataConnection.on("data", (data) => {
                console.log("Received packet");
                console.log(data);
                packet = data;
                //console.log(packet.length)
            })
            /*pong = setInterval(() => {
                dataConnection.send("poll");
            }, 7000)*/
            peer.on("error", (err) => {
                console.log(err.type);
            }) 
        })

    }

    function formatTimeLabel(raw) {
        if (packet.event.format == "AM/PM") {
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
</script>
<svelte:head>
    <title>Live Display | {eventName}</title>
</svelte:head>
<style>
    form {
        background-color: lightgrey;
        margin: 40px;
        padding: 10px;
        border-radius: 50px;
        input {
            border-radius: 40px;
            padding: 10px
        }
    }
    #event {
        padding: 5px;
        background-color: rgba(94, 93, 93, 0.671);
        border-radius: 35px;
        top: 55px;
        width: 60%;
        margin-left: 20%;
        margin-right: 20%;
        text-align: center;

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

    .field {
        margin-left: 20px;
        margin-right: 20px;
        margin-top: 40px;
    }

    #announcementBox {
        padding: 10px;
        background-color: grey;
        color: white;
        border-radius: 20px;

        h2 {
            font-size: 20px;
            margin-top: 10px;
        }
        p {
            color: white;
        }
    }
</style>
{#if mode == 0}
<div out:slide in:slide={{delay:1000}} style:margin-top=20px>
    <h1>{eventName} Jumbotron</h1>
    <p style:margin-bottom=40px>Let's get you started! Please enter the 5-digit code provided by your event organizers in the form below.</p>
    <form onsubmit={connectToEvent}>
        <input bind:value={inputtedId} type="text" minlength=5 maxlength=5 required placeholder="ID Number"><br>
    </form>
</div>
{/if}
{#if mode == 1}
<div in:fly={{y:-200, delay:1000}} out:fly={{y:-200}} class="field">
    <p>Welcome to</p>
    <h1 style:margin-top=0px style:margin-bottom=40px>{eventName} {packet.city}</h1>
</div>
{#if packet.event != null}
<div id="event" in:slide={{delay: 2000}} out:slide class="field">
    <p><strong>{packet.event.title}</strong> <span id="timeLabel">{formatTimeLabel(packet.event.time)}</span></p>
</div>
{/if}
{#if packet.announcement != null}
<div in:slide={{delay: 2000}} out:slide class="field">
    <div id="announcementBox">
        <h2>{packet.announcement.title}</h2>
        <p>{packet.announcement.message}</p>
    </div>
</div>
{/if}
{#if packet.presentation != null}
<div in:slide={{delay: 2000}} out:slide class="field">
    <h2>A presentation is currently being displayed on the Jumbotron</h2>
    <p>You can follow along by clicking the button</p>
    <p><a href={packet.presentation} target="_blank"><button>View Content</button></a></p>
</div>
{/if}
{#if packet.noContent != null}
<div in:slide={{delay: 2000}} out:slide class="field">
    <p>No content is currently being displayed on the Jumbotron.</p>
</div>
{/if}
{/if}