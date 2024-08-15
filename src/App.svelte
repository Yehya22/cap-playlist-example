<main>
    <Player />
    <Toaster containerClassName={Capacitor.getPlatform() === 'ios' ? 'ios-margin-top' : ''} />
</main>

<script>
import {Capacitor} from '@capacitor/core'
import Player from './Player.svelte'
import {Toaster} from 'svelte-french-toast'
</script>

<style>
.ios-margin-top {
    margin-top: 30px;
}
</style>
