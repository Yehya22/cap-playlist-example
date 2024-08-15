<main>
  <Player />
  <Toaster containerStyle="margin-top: {Capacitor.getPlatform() === 'ios' ? '40px' : '0'}" />
</main>

<script>
import {Capacitor} from '@capacitor/core'
import Player from './Player.svelte'
import {Toaster} from 'svelte-french-toast'
</script>