<template>
  <Transition>
    <div :key="currentLyrics?.[0]?.startTime" :class="[
      'lyric-am',
      {
        'lyric-left': setting.lyricsPosition === 'left',
        'lyric-center': setting.lyricsPosition === 'center',
        'loading': music.getLoadingState
      }
    ]">
      <LyricPlayer ref="lyricPlayerRef" :lyricLines="currentLyrics" :currentTime="music.getPlaySongTime.currentTime * 1000"
        :playing="music.getPlayState"
        :alignAnchor="setting.lyricsBlock === 'center' ? 'center' : 'top'"
        :alignPosition="setting.lyricsBlock === 'center' ? 0.5 : 0.2" :enableSpring="setting.showYrcAnimation"
        :enableScale="setting.showYrcAnimation" :enableBlur="setting.lyricsBlur"
        :wordFadeWidth="0.5" :linePosXSpringParams="setting.springParams.posX"
        :linePosYSpringParams="setting.springParams.posY" :lineScaleSpringParams="setting.springParams.scale" :style="{
          '--amll-lyric-view-color': setting.immersivePlayer ? mainColor : 'rgb(239, 239, 239)',
          '--amll-lyric-player-font-size': setting.lyricsFontSize * 10 + 'px',
          '--amll-lyric-player-line-height': setting.lyricLineHeight,
          'font-weight': setting.lyricFontWeight,
          'font-family': setting.lyricFont,
          'letter-spacing': setting.lyricLetterSpacing
        }" class="am-lyric" @line-click="(e) => {
          if (music.getPlayState) lrcTextClick(e.line.getLine().startTime)
        }" />
    </div>
  </Transition>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { musicStore, settingStore, siteStore } from "@/store";
import { LyricPlayer } from "@applemusic-like-lyrics/vue";

const music = musicStore();
const setting = settingStore();
const site = siteStore();

const lyricPlayerRef = ref(null);

onMounted(() => {
  console.log(music.getPlaySongLyric);
});

// 歌词主色
const mainColor = computed(() => {
  return site.songPicColor || 'rgb(239, 239, 239)';
});

// Get current lyrics based on settings
const currentLyrics = computed(() => {
  const songLyric = music.songLyric || { lrcAMData: [], yrcAMData: [] };
  return setting.showYrc && songLyric.yrcAMData?.length
    ? songLyric.yrcAMData
    : songLyric.lrcAMData || [];
});

const emit = defineEmits(["lrcTextClick"]);

// Handle line click
const lrcTextClick = (time) => {
  if (time != null) {
    emit("lrcTextClick", time / 1000);
    music.setPlayState(true);
  }
};
</script>

<style lang="scss" scoped>
.lyric-am {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
  filter: drop-shadow(0px 4px 6px rgba(0, 0, 0, 0.2));
  mask: linear-gradient(180deg,
      hsla(0, 0%, 100%, 0) 0,
      hsla(0, 0%, 100%, 0.6) 5%,
      #fff 10%,
      #fff 75%,
      hsla(0, 0%, 100%, 0.6) 85%,
      hsla(0, 0%, 100%, 0));
  opacity: 1;
  transform: translateZ(0) scale(1);
  will-change: transform, opacity;
  transition: transform 0.5s cubic-bezier(0.34, 1.56, 0.64, 1),
              opacity 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);

  &.loading {
    opacity: 0;
    transform: scale(0.8);
  }

  &.lyric-left {
    :deep(.am-lyric) {
      text-align: left;

      div {
        transform-origin: left center;
      }
    }
  }

  &.lyric-center {
    :deep(.am-lyric) {
      text-align: center;

      div {
        transform-origin: center;
      }
    }
  }

  :deep(.am-lyric) {
    width: 100%;
    height: 100%;
    position: absolute;
    left: 0;
    top: 0;
    font-synthesis: none;
    text-rendering: optimizeLegibility;
  }
}

.v-enter-active,
.v-leave-active {
  transition: opacity 0.3s ease, transform 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
  transform: scale(0.8);
}
</style>
