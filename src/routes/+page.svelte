<script>
    import { base } from "$app/paths";
</script>
<style>
    * {
        margin-left: 30px;
        margin-right: 30px;
    }
</style>
<svelte:head>
    <title>Jumbotron</title>
</svelte:head>

<h1>Welcome to Jumbotron</h1>
<p>This is a work in progress, since specific theming and art for V5 has not been released. It doesn't have all the bells and whistles; however, the current stage of this project represents the functionality of the backbone and critical systems. You can check out the repository <a style:margin=0px href="https://github.com/lynn89-eefje/jumbotron">here</a></p>

<h2>Other pages</h2>
<p>{"/[city]/control"}</p>
<p>{"/[city]/display"}</p>
<p>{"/live/"}</p>

<h2>Testing</h2>
<p><a href={base+"/leafyland/control"}>Control Window</a></p>
<p><a href={base+"/live"}>Liveshare</a></p>