<section class="player-wrapper">
    <div class="player-cont">
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
import toast from 'svelte-french-toast'

const platform = Capacitor.getPlatform()

let tracks = $state([
    {
        isStream: true,
        trackId: '1',
        assetUrl:
            'https://ia801809.us.archive.org/8/items/homeacre_2408_librivox/homeacre_01_roe_128kb.mp3',
        title: 'CHAPTER I TREE-PLANTING - Part 1',
        album: 'CHAPTER I TREE-PLANTING - Part 1',
    },
    {
        isStream: true,
        trackId: '2',
        assetUrl:
            'https://www.archive.org/download/homeacre_2408_librivox/homeacre_02_roe_128kb.mp3',
        title: 'CHAPTER I TREE-PLANTING - Part 2',
        album: 'CHAPTER I TREE-PLANTING - Part 2',
    },
    // {
    //     isStream: true,
    //     trackId: '3',
    //     assetUrl:'https://www.archive.org/download/homeacre_2408_librivox/homeacre_03_roe_128kb.mp3',
    //     title: 'CHAPTER II FRUIT-TREES AND GRASS – Part 1',
    //     album: 'CHAPTER II FRUIT-TREES AND GRASS – Part 1',
    // },
    // {
    //     isStream: true,
    //     trackId: '4',
    //     assetUrl:'https://www.archive.org/download/homeacre_2408_librivox/homeacre_04_roe_128kb.mp3',
    //     title: 'CHAPTER II FRUIT-TREES AND GRASS – Part 2',
    //     album: 'CHAPTER II FRUIT-TREES AND GRASS – Part 2',
    // },
    // {
    //     isStream: true,
    //     trackId: '5',
    //     assetUrl:'https://www.archive.org/download/homeacre_2408_librivox/homeacre_05_roe_128kb.mp3',
    //     title: 'CHAPTER III THE GARDEN – Part 1',
    //     album: 'CHAPTER III THE GARDEN – Part 1',
    // },
])

let reader_state = {
    src: 1,
}

function fmt_time(seconds) {
    const minutes = Math.floor(seconds / 60)
    const secs = Math.floor(seconds % 60)
    return `${minutes}:${secs < 10 ? '0' : ''}${secs}`
}

onMount(() => {
    if (tracks.length) {
        set_tracks(tracks, reader_state.src)
    }
})

let previous_tracks
let previous_track_id

$effect(() => {
    if (tracks !== previous_tracks || reader_state.src !== previous_track_id) {
        if (tracks.length) {
            set_tracks(tracks, reader_state.src)
        }
        previous_tracks = tracks
        previous_track_id = reader_state.src
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
            break
        case RmxAudioStatusMessage.RMXSTATUS_PLAYING:
            playing = true
            toast.success('Playing')
            console.log('Playing')
            break
        case RmxAudioStatusMessage.RMXSTATUS_DURATION:
            duration = value.duration
            toast.success('Duration updated')
            console.log('Duration updated')
            break
        case RmxAudioStatusMessage.RMXSTATUS_PAUSE:
            playing = false
            toast.success('Paused')
            console.log('Paused')
            break
        case RmxAudioStatusMessage.RMXSTATUS_SEEK:
            toast.success('Seek')
            console.log('Seek')
            break
        case RmxAudioStatusMessage.RMXSTATUS_COMPLETED:
            toast.success('Completed')
            console.log('Completed')
            ended()
            break
    }

    if (platform === 'ios' || platform === 'web') {
        // RmxAudioStatusMessage.RMXSTATUS_<PLAYING|DURATION> events aren't reliable on iOS
        playing = value?.status === 'playing'
        duration = value?.duration || 0
    }
}
rmx_player.on('status', event_cb)

async function set_tracks(tracks, track_id) {
    try {
        await rmx_player.setPlaylistItems($state.snapshot(tracks), {
            playFromId: `${track_id}`,
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

function ended() {
    if (reader_state.src === tracks.length) {
        setTimeout(async () => {
            reader_state.src = 1
            await rmx_player.playTrackById(`${reader_state.src}`)
        }, 100)
    } else {
        reader_state.src += 1
    }
}

onDestroy(() => {
    rmx_player.off('status', event_cb)
    rmx_player.pause()
    rmx_player.clearAllItems()
})
</script>

<style>
.player-wrapper {
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
}
.player-cont {
    direction: ltr;
    width: 100%;
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
