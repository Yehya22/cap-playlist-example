<div>
    <div>
        <label>Filter by:</label>
        <select on:change={e => filterLogs(e.target.value)}>
            <option value="all">All</option>
            {#each eventTypes as type}
                <option value={type}>{type}</option>
            {/each}
        </select>
    </div>
    <pre>
        {#each filteredLogs as log}
            {log.type}: {log.message}
        {/each}
    </pre>
</div>

<script>
import {onMount} from 'svelte'
import {rmx_player} from './Player.svelte'

let logs = []
let filter = 'all'

const eventTypes = ['error', 'buffering', 'seek']

function logEvent(event) {
    logs = [...logs, event]
}

function filterLogs(type) {
    filter = type
}

onMount(() => {
    rmx_player.on('error', e => logEvent({type: 'error', message: e.message}))
    rmx_player.on('buffering', e => logEvent({type: 'buffering', message: 'Buffering...'}))
    rmx_player.on('seek', e => logEvent({type: 'seek', message: `Seeked to ${e.position}`}))
})

$: filteredLogs = filter === 'all' ? logs : logs.filter(log => log.type === filter)
</script>
