<section class="player-cont">
    <input
        type="range"
        min="0"
        max={duration || 1}
        step="any"
        oninput={e => seek_to(e.target.value)}
        onclick={window._useragent?.ios
            ? e => seek_to((e.offsetX / e.target.offsetWidth) * duration)
            : () => {}}
        bind:value={seeker_value}
        style="--bg-size: {(time_passed / (duration || 1)) * 100 || 0}%"
    />
    <div class="now-playing">
        <span class="time-passed">{fmt_time(time_passed)}</span>
        <PlayButton on:click={() => rmx_player[playing ? 'pause' : 'play']()} {playing} />
        <span class="time-remaining">-{fmt_time(duration - time_passed)}</span>
    </div>
</section>

<svelte:window
    onkeydown={e => {
        const active_tag = document.activeElement.tagName
        if (e.key === ' ' && ['INPUT', 'TEXTAREA', 'BUTTON'].indexOf(active_tag) === -1) {
            rmx_player[playing ? 'pause' : 'play']()
        }
    }}
/>

<script context="module">
import {RmxAudioPlayer, RmxAudioStatusMessage, Playlist} from 'capacitor-plugin-playlist'

export const rmx_player = new RmxAudioPlayer()
rmx_player.initialize()
Playlist.setOptions({verbose: false, options: {icon: 'app'}})
</script>

<script>
import PlayButton from './PlayButton.svelte'
import {Capacitor} from '@capacitor/core'
import {onDestroy, onMount} from 'svelte'

const platform = Capacitor.getPlatform()
let {tracks = [], track_id, positions, timeupdate, on_ended} = $props()

function fmt_time(seconds) {
    const minutes = Math.floor(seconds / 60)
    const secs = Math.floor(seconds % 60)
    return `${minutes}:${secs < 10 ? '0' : ''}${secs}`
}
onMount(() => {
    if (tracks.length) {
        set_tracks(tracks, track_id, positions)
    }
})

let previous_tracks
let previous_track_id

$effect(() => {
    if (tracks !== previous_tracks || track_id !== previous_track_id) {
        if (tracks.length) {
            set_tracks(tracks, track_id)
        }
        previous_tracks = tracks
        previous_track_id = track_id
    }
})

let time_passed = $state(0)
let seeker_value = $state(0)
let duration = $state(0)
let playing = $state(false)

function event_cb(data) {
    const {msgType, value} = data

    switch (msgType) {
        case RmxAudioStatusMessage.RMXSTATUS_PLAYBACK_POSITION:
            time_passed = value.currentPosition
            seeker_value = value.currentPosition
            timeupdate(value.currentPosition)
            break
        case RmxAudioStatusMessage.RMXSTATUS_PLAYING:
            if (platform === 'android') playing = true
            break
        case RmxAudioStatusMessage.RMXSTATUS_DURATION:
            if (platform === 'android') duration = value.duration
            break
        case RmxAudioStatusMessage.RMXSTATUS_PAUSE:
            if (platform === 'android') playing = false
            break
        case RmxAudioStatusMessage.RMXSTATUS_SEEK:
            console.log('seek')
            break
        case RmxAudioStatusMessage.RMXSTATUS_COMPLETED:
            on_ended()
            break
    }

    if (platform === 'ios' || platform === 'web') {
        // RmxAudioStatusMessage.RMXSTATUS_<PLAYING|DURATION> events aren't reliable on iOS
        playing = value?.status === 'playing'
        duration = value?.duration || 0
    }
}
rmx_player.on('status', event_cb)

const set_tracks = async (tracks, track_id) => {
    try {
        await rmx_player.setPlaylistItems($state.snapshot(tracks), {
            playFromId: `${track_id}`,
            playFromPosition: Math.max(0, positions[track_id] || 0),
            startPaused: !playing,
        })
    } catch (error) {
        console.error('Error setting playlist items:', error)
    }
}

async function seek_to(position) {
    position = Math.trunc(position) // android
    await rmx_player.seekTo(position)
    seeker_value = position
}

onDestroy(() => {
    rmx_player.off('status', event_cb)
    rmx_player.pause()
    rmx_player.clearAllItems()
})
</script>

<style>
.player-cont {
    direction: ltr;
    padding: 0.5rem;
    box-shadow: 0 2px 1px 0 whitesmoke;

    background: #fff;
    box-shadow: 0 -3px 1px 0 #f5f5f5;
}

/* Seeker */
input[type='range'] {
    display: block;
    width: 100%;
    border: 0;
    margin: 0;
    height: 20px;
    background: transparent;
    cursor: pointer;
    outline: 0;
    -webkit-appearance: none;
    --bg-size: 1%;
}
input[type='range']::-webkit-slider-runnable-track {
    height: 9px;
    border-radius: 5px;
    border: solid 1px #eee;
    background:
        linear-gradient(#016982, #016982) no-repeat,
        linear-gradient(#ccc, #ccc);
    background-size: var(--bg-size) 100%;
}
input[type='range']::-webkit-slider-thumb {
    -webkit-appearance: none;
    margin-top: -5px;
    width: 16px;
    height: 16px;
    border-radius: 50%;
    background: #fff;
    box-shadow: 0 0 6px #aaa;
    border: 0; /* iOS */
}
input[type='range']::-moz-range-thumb {
    margin-top: -5px;
    width: 16px;
    height: 16px;
    border-radius: 50%;
    background: #fff;
    box-shadow: 0 0 6px #aaa;
    border: 0; /* Fx */
}

input[type='range']::-moz-range-progress {
    background-color: #016982;
}
input[type='range']::-moz-range-track {
    background-color: #ccc;
}
input[type='range']::-moz-focus-outer {
    border: 0;
}
input[type='range']::-ms-fill-upper {
    background-color: #ccc;
}
input[type='range']::-ms-fill-lower {
    background-color: #016982;
}

/* Times & play button */
.now-playing {
    display: flex;
    text-align: center;
    padding: 0.3rem 0;
}
.time-passed,
.time-remaining {
    color: #aaa;
    font-family: sans-serif;
    font-size: 0.7rem;
    flex: 1;
}
</style>
