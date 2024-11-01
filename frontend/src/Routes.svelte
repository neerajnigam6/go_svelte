<script>
    import Router from 'svelte-spa-router'
    import Video from './lib/Screens/Video.svelte';

    let routes = {
        '/video/:id': Video
    }
</script>

<Router {routes} />