<script>
    import { base } from "$app/paths";
</script>

<svelte:head>
    <title>Jumbotron</title>
</svelte:head>

<h1>Welcome to Jumbotron</h1>
<p>This is a work in progress. You can check out the repository <a href="https://github.com/lynn89-eefje/jumbotron">here</a></p>

<h2>Other pages</h2>
<p>{"/[city]/control"}</p>
<p>{"/[city]/display"}</p>
<p>{"/live/"}</p>

<h2>Testing</h2>
<p><a href={base+"/leafyland/control"}>Control Window</a></p>