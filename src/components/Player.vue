<template>
  <div id="player">

    <!--
      Audio control in HTML5 is done primarily through the HTMLMediaElement API, which is a little awkward to use
      with Vue. We can't bind to it directly, and Vue won't pick up changes on it through dependency detection.
      To get the interactivity we need, we have to bind callbacks here as well as keep a copy of any properties
      we want Vue to react to. (see audioPlayerState below)
    -->
    <audio @durationchange="updateDuration" @ended="onAudioEnd" @timeupdate="updateTime" ref='audioElement'></audio>

    <div id="now-playing">

      <div id="album-art-container">
        <img
            alt="Album art"
            title="music from games you used to play"
            id="album-art"
            sizes="(max-width: 320px) 320px, 512px"
            src="img/album-art.png"
            srcset="img/album-art-320px.png 320w,
                    img/album-art.png 512w"
        >
      </div>

      <PlaybackControls
          :audioElementState="audioElementState"
          @play-toggle="playToggle"
          @rewind="rewind"
          @seek-to="seekTo"
          @skip="skip"
      />

    </div>

    <div id="song-list">
      <template v-for="song in songs">
        <SongListItem :nowPlaying="isSongPlaying(song)" :song="song" @click="songClicked"/>
      </template>
    </div>

  </div>
</template>

<script lang="ts">
  import {Component, Vue} from 'vue-property-decorator';
  import songs, {Song} from '@/assets/songs';
  import SongListItem from '@/components/SongListItem.vue';
  import PlaybackControls from '@/components/PlaybackControls.vue';

  export interface AudioElementState
  {
    duration: number;
    currentTime: number;
    currentSrc: string | null;
    isPlaying: boolean;
  }

  enum SkipDirection { Forward, Backward }

  const MOVE_TO_PREV_TRACK_THRESHOLD = 2; // seconds

  @Component({
    components: {PlaybackControls, SongListItem}
  })
  export default class Player extends Vue
  {
    // need this declaration to expose the imported songs to the template
    songs = songs;

    // Assert that this will not be null, since we only ever use it after
    // the mounted() callback
    audioElement!: HTMLAudioElement;

    // In order to fully integrate with Vue's dependency detection, we need to
    // keep a copy of some of the HTMLAudioElement's state here and push it down
    // to the PlaybackControls. It's not the nicest looking process, but there's
    // no way to get Vue to set up its hooks on an HTMLElement as far as I'm
    // aware.
    audioElementState: AudioElementState = {
      isPlaying: false,
      duration: 0,
      currentTime: 0,
      currentSrc: null,
    };

    mounted()
    {
      // This helps with type inference, otherwise we would have to repeat the cast
      // everywhere that we would access $refs
      this.audioElement = this.$refs.audioElement as HTMLAudioElement;
    }

    songClicked(event: Song)
    {
      this.audioElement.src = event.url;
      this.audioElement.play();
      this.audioElementState.isPlaying = true;
      this.audioElementState.currentSrc = event.url;
    }

    isSongPlaying(song: Song)
    {
      return this.audioElementState.currentSrc === song.url;
    }

    updateDuration()
    {
      this.audioElementState.duration = this.audioElement.duration;
    }

    updateTime()
    {
      this.audioElementState.currentTime = this.audioElement.currentTime;
    }

    playToggle()
    {
      if (this.audioElement.paused || this.audioElement.ended)
      {
        if (this.audioElement.ended)
        {
          this.audioElement.currentTime = 0;
        }

        this.audioElement.play();
        this.audioElementState.isPlaying = true;
      }
      else
      {
        this.audioElement.pause();
        this.audioElementState.isPlaying = false;
      }
    }

    seekTo(positionFraction: number)
    {
      this.audioElement.currentTime = this.audioElement.duration * positionFraction;
    }

    skip(direction = SkipDirection.Forward)
    {
      let currentIndex = songs.findIndex(song => song.url === this.audioElementState.currentSrc);
      let nextIndex = direction === SkipDirection.Forward ? (currentIndex + 1) % songs.length : currentIndex - 1;

      if (nextIndex < 0)
      {
        nextIndex = songs.length - 1;
      }

      let nextSong = songs[nextIndex];

      this.audioElement.src = nextSong.url;
      this.audioElement.play();
      this.audioElementState.isPlaying = true;
      this.audioElementState.currentSrc = nextSong.url;
    }

    rewind()
    {
      if (this.audioElement.currentTime < MOVE_TO_PREV_TRACK_THRESHOLD)
      {
        this.skip(SkipDirection.Backward);
      }
      else
      {
        this.audioElement.currentTime = 0;
      }
    }

    onAudioEnd()
    {
      this.audioElementState.isPlaying = false;
    }
  }
</script>

<style lang="scss" scoped>
  $layout-breakpoint: 900px;

  #player
  {
    display:   flex;
    flex-flow: column;

    #now-playing
    {
      display: flex;
      flex-flow: column;
    }

    #song-list
    {
      display:   flex;
      flex-flow: column;
      flex-grow: 0;
    }

    @media only screen and (min-width: $layout-breakpoint)
    {
      flex-flow: row;
      align-items: center;
      justify-content: center;
      height: 100vh;

      #song-list
      {
        margin-left: 64px;
        min-width: 250px;
      }
    }
  }

  #album-art-container
  {
    max-width: 512px;
    margin:    auto;

    #album-art
    {
      max-width: 100%;
    }
  }
</style>
