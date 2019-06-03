<template>

  <div :class="{'song-list-item': true, 'now-playing': nowPlaying}">
    <p @click="handleClick">{{song.name}}</p>
  </div>

</template>

<script lang="ts">
  import {Component, Prop, Vue} from 'vue-property-decorator';
  import {Song} from '@/assets/songs';

  @Component
  export default class SongListItem extends Vue
  {
    @Prop() song!: Song;
    @Prop() nowPlaying = false;

    handleClick()
    {
      this.$emit('click', this.song);
    }
  }
</script>

<style lang="scss" scoped>

  %selected
  {
    background-color: #576ca8;
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
</style>
