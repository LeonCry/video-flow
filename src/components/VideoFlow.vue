<script setup lang="ts">
import mpegts from 'mpegts.js';

const props = defineProps<{
  url: string
}>();

const liveLoaded = ref(false);

let player: mpegts.Player | null;
let lastDecodedFrames = 0; // 10s前的解码帧数
let currentDecodedFrames = 0; // 当前解码帧数
const videoElement = ref<HTMLMediaElement | null>(null);

function initPlay() {
  if (!props.url || !videoElement.value) {
    return;
  }
  lastDecodedFrames = 0;
  currentDecodedFrames = 0;
  player = mpegts.createPlayer(
    {
      type: 'flv',
      isLive: true,
      hasAudio: false,
      url: props.url,
    },
    {
      liveBufferLatencyChasing: true,
    },
  );
  player.attachMediaElement(videoElement.value);
  player.load();
  player.on(mpegts.Events.MEDIA_INFO, () => {
    liveLoaded.value = true;
    console.log('视频开始播放');
  });
  player.on(mpegts.Events.ERROR, (e) => {
    console.log('视频加载错误', e);
    reloadPlayer();
  });
  player.on(mpegts.Events.RECOVERED_EARLY_EOF, () => {
    console.log('视频播放结束');
  });
  player.on(mpegts.Events.STATISTICS_INFO, (e) => {
    // console.log("视频播放信息", e.decodedFrames);
    const frame = e.decodedFrames || 0;
    currentDecodedFrames = frame;
  });
  // player.play();
  setTimeout(() => {
    console.log('视频play()触发');
  }, 50);
}

const interval = setInterval(() => {
  if (currentDecodedFrames === lastDecodedFrames) {
    console.log(
      `视频播放卡顿，10s前解码帧数为 ${
        lastDecodedFrames
      } ，当前解码帧数为 ${
        currentDecodedFrames}`,
    );
    if (videoElement.value) {
      reloadPlayer();
    }
  }
  lastDecodedFrames = currentDecodedFrames;
}, 15000);

function destroyPlayer() {
  if (player) {
    console.log('视频判断需要销毁');
    player!.pause();
    player!.unload();
    player!.detachMediaElement();
    console.log('视频播放器销毁');
    player!.destroy();
    player = null;
  }
}

onMounted(() => {
  initPlay();
});

function reloadPlayer() {
  liveLoaded.value = false;
  destroyPlayer();
  setTimeout(() => {
    initPlay();
  }, 100);
}

// 切换播放url
watch(
  () => props.url,
  () => {
    if (props.url) {
      reloadPlayer();
    }
  },
  {
    deep: true,
  },
);

watch(videoElement, (val) => {
  if (!val) {
    destroyPlayer();
    clearInterval(interval);
  }
});

onBeforeUnmount(() => {
  destroyPlayer();
  clearInterval(interval);
});
</script>

<template>
  <div
    v-loading="!liveLoaded"
    element-loading-background="rgba(0, 0, 0, 0.33)"
    element-loading-text="加载中..."
    class="w-full h-full"
  >
    <video
      ref="videoElement"
      autoplay
      muted
      loop
      class="video-js video-content"
    >
      <source :src="props.url">
    </video>
  </div>
</template>

<style scoped lang="scss">
.video-content {
  width: 100%;
  height: 100%;
  background-color: transparent !important;
  object-fit: fill;
}
</style>
