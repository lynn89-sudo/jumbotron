<script>
    import { base } from "$app/paths";
    import { page } from "$app/state";
    import { onMount } from "svelte";
    import { slide } from "svelte/transition";

    import { eventName } from "$lib/event.js";
    import { proccessCity } from "$lib/event.js";

    import { sync } from "$lib/sync.svelte.js";
    import { tutorial } from "$lib/sync.svelte.js";

    import Peer from "peerjs";
    import DataConnection from "peerjs";

    let liveshareEnabled = $state(false);
    let blockConnections = $state(false);
    let blockPings = $state(false);

    let peer;
    let tries = 3;
    let peerInfo = $state({});

    function enableLiveshare() {
        let code = codeGen();
        peer = new Peer("hackclub" + eventName + code);
        peer.on("error", function(err) {
            if (err.type = "peer-unavailable" && tries > 0) {
                tries--;
                peer.destroy();
                enableLiveshare();
            }
            else if (err.type != "peer-unavailable") {
                window.alert(`PeerJS Error: ${err.type}. You should try restarting liveshare. If this issue persists, copy the error and contact support.`)
            }
        })
        peer.on("open", () => {
            console.log("Peer has connected");
            //console.log(peer);
            liveshareEnabled = true;
            peerInfo.code = code;
            peerInfo.connections = 0;
            peerInfo.packetsSent = 0;
            //console.log(peerInfo);
        })
        peer.on("connection", (dataConnection) => {
            dataConnection.on("open", () => {
                if (blockConnections) {
                    dataConnection.close();
                    peerInfo.connections++;
                }
                else {
                    dataConnection.send(sendPacket());
                    peerInfo.connections++;
                }
            })
            dataConnection.on("close", () => {
                peerInfo.connections--;
            })
        })
    }
    function destroyLiveshare() {
        peer.destroy();
        peer = null;
        console.log("Peer destroyed");
        liveshareEnabled = false;
        peerInfo = {};
    }
    /*
    onMount(() => {
        enableLiveshare();
    })
        */
    
    function codeGen() {
        let code = "";
        for (let i = 0; i < 5; i++) {
            let rand = Math.random()*10
            rand  = Math.floor(rand);
            code += "" + rand;
        }
        return code;
    }

    function sendPacket() {
        let packet = {};
        packet.city = proccessCity(page.params.city);
        packet.event = {};
        packet.event.label = localStorage.getItem("jumbotron.event.label");
        packet.event.time = localStorage.getItem("jumbotron.event.time");
        peerInfo.packetsSent++;
        return packet;
    }

    async function copyCode() {
        await navigator.clipboard.writeText(peerInfo.code);   
    }

    function toggle(which) {
        if (which == "connections") {
            if (blockConnections) {
                blockConnections = false;
            }
            else {
                blockConnections = true;
            }
        }
        else if (which == "pings") {
            if (blockPings) {
                blockPings = false;
            }
            else {
                blockPings = true;
            }
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

    button.copy:active {
        background-color: white;
        transform: scale(1.04);
        span {
            color: rgb(85, 85, 85);
        }
    }
    h4 {
        margin-top: 35px;
    }

    span.monitor {
        padding: 8px;
        margin: 3px;
        border-radius: 20px;
        background-color: grey;
        color: white;
    }
</style>

<div>
    {#if tutorial.enabled}
    <p>Liveshare allows you to share information displayed on Jumbotron to your participants through their own screens.</p>
    {/if}
    {#if !liveshareEnabled}
    <p><button onclick={enableLiveshare}>Enable Liveshare</button></p>
    <p><i>If you are using a low performance device, Liveshare is not recomended.</i></p>
    {:else}
    <p><button onclick={destroyLiveshare}>Disable Liveshare</button></p>
    <h4>Participant Code</h4>
    <h5 style:font-size=30px>{peerInfo.code} <button onclick={copyCode} class="copy"><span translate="no" class="material-symbols-outlined">file_copy</span></button></h5>
    {#if tutorial.enabled}<p>Participants should use this code to view information on their device; you might want to copy this code into an announcement. If your liveshare restarts, this code will likely change.</p>{/if}
    <h4>Monitor</h4>
    <p>Current connections: <span class="monitor">{peerInfo.connections}</span></p>
    <p>Information pings: <span class="monitor">{peerInfo.packetsSent}</span></p>
    <p>{#if !blockConnections}<button onclick={() => {toggle("connections")}}>Block New Connections</button>{:else}<button onclick={() => {toggle("connections")}}>Unblock New Connections</button>{/if} {#if !blockPings}<button onclick={() => {toggle("pings")}}>Block Information Pings</button>{:else}<button onclick={() => {toggle("pings")}}>Unblock Information Pings</button>{/if}</p>
    {/if}
</div>
