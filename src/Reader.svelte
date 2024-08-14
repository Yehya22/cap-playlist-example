<Player
    {tracks}
    track_id={reader_state.src}
    on_ended={ended}
    {timeupdate}
    positions={reader_state.times}
/>

<script>
import Player from './Player.svelte'
import {rmx_player} from './Player.svelte'

let tracks = [
    {
        isStream: true,
        trackId: 1,
        assetUrl:
            'https://ia801809.us.archive.org/8/items/homeacre_2408_librivox/homeacre_01_roe_128kb.mp3',
        title: 'CHAPTER I TREE-PLANTING - Part 1',
        album: 'CHAPTER I TREE-PLANTING - Part 1',
    },
    {
        isStream: true,
        trackId: 2,
        assetUrl:
            'https://www.archive.org/download/homeacre_2408_librivox/homeacre_02_roe_128kb.mp3',
        title: 'CHAPTER I TREE-PLANTING - Part 2',
        album: 'CHAPTER I TREE-PLANTING - Part 2',
    },
    {
        isStream: true,
        trackId: 3,
        assetUrl:
            'https://www.archive.org/download/homeacre_2408_librivox/homeacre_03_roe_128kb.mp3',
        title: 'CHAPTER II FRUIT-TREES AND GRASS – Part 1',
        album: 'CHAPTER II FRUIT-TREES AND GRASS – Part 1',
    },
    {
        isStream: true,
        trackId: 4,
        assetUrl:
            'https://www.archive.org/download/homeacre_2408_librivox/homeacre_04_roe_128kb.mp3',
        title: 'CHAPTER II FRUIT-TREES AND GRASS – Part 2',
        album: 'CHAPTER II FRUIT-TREES AND GRASS – Part 2',
    },
    {
        isStream: true,
        trackId: 5,
        assetUrl:
            'https://www.archive.org/download/homeacre_2408_librivox/homeacre_05_roe_128kb.mp3',
        title: 'CHAPTER III THE GARDEN – Part 1',
        album: 'CHAPTER III THE GARDEN – Part 1',
    },
]

let reader_state = {
    src: 1,
    times: {},
}

let is_ended = $state(false)

function timeupdate(position) {
    reader_state.times[reader_state.src] = position
}

function ended() {
    if (is_ended) return
    is_ended = true

    reader_state.times[reader_state.src] = 0

    if (reader_state.src === tracks.length) {
        setTimeout(() => {
            console.log('ended')
            rmx_player.pause()
        }, 100)
        reader_state.src = 1
    } else {
        reader_state.src += 1
    }

    is_ended = false
}
</script>
