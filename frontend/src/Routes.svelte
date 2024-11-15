<script>
    import Router from 'svelte-spa-router'
    import VideoScreen from './lib/Screens/VideoScreen.svelte';

    let routes = {
        '/video/:id': VideoScreen
    }
</script>

<Router {routes} />