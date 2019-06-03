<template>
  <div id="playback-controls">

    <progress :value="currentProgress" @click="seekTo"></progress>

    <div id="control-buttons">
      <button :disabled="!isSongSelected" @click="$emit('rewind')" type="button">
        <img src="img/seek-backward.svg">
      </button>
      <button :disabled="!isSongSelected" @click="$emit('play-toggle')" type="button">
        <img :src="playIconUrl">
      </button>
      <button :disabled="!isSongSelected" @click="$emit('skip')" type="button">
        <img src="img/seek-forward.svg">
      </button>
    </div>

  </div>
</template>

<script lang="ts">
  import {Component, Prop, Vue} from 'vue-property-decorator';
  import {AudioElementState} from '@/components/Player.vue';

  @Component
  export default class PlaybackControls extends Vue
  {
    @Prop()
    audioElementState!: AudioElementState;

    get isSongSelected()
    {
      return this.audioElementState.currentSrc !== null;
    }

    get currentProgress()
    {
      if (this.audioElementState.duration === 0)
      {
        return 0;
      }

      return this.audioElementState.currentTime / this.audioElementState.duration;
    }

    get playIconUrl()
    {
      return this.audioElementState.isPlaying ? 'img/pause.svg' : 'img/play.svg';
    }

    seekTo(event: MouseEvent)
    {
      let progressBarElement = event.target as HTMLProgressElement;
      this.$emit('seek-to', event.offsetX / progressBarElement.clientWidth);
    }
  }
</script>

<style lang="scss">

  %selected
  {
    background-color: #576ca8;
  }

  #control-buttons
  {
    margin:
    {
      top:    16px;
      bottom: 16px;
    }
  }

  button
  {
    background: none;
    border:     none;
    color:      white;
    cursor:     pointer;

    &[disabled]
    {
      opacity: 0.2;
      cursor:  default;
    }

    &:hover
    {
      path
      {
        fill: #576ca8;
      }
    }

    img
    {
      width:          48px;
      height:         48px;
      vertical-align: middle;
    }
  }

  .song-list-item
  {
    padding:
    {
      top:    8px;
      bottom: 8px;
    }

    cursor: pointer;

    &:hover
    {
      @extend %selected;
    }
  }

  .now-playing
  {
    @extend %selected;
  }

  progress
  {
    border:             0;
    margin:             0;
    padding:            0;
    background-color:   rgba(255, 255, 255, 0.2);
    height:             4px;
    -webkit-appearance: none;
    color:              white;

    display:            block;
    width:              100%;

    cursor:             pointer;

    &::-moz-progress-bar
    {
      background-color: white;
    }

    &::-webkit-progress-bar
    {
      background-color: rgba(255, 255, 255, 0.2);
    }

    &::-webkit-progress-value
    {
      background: white;
    }

  }
</style>
