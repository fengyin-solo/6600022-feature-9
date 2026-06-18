<template>
  <div class="bg-gray-900 rounded-xl p-4 border border-gray-700">
    <h3 class="text-lg font-bold text-green-400 mb-3">棋谱回放</h3>

    <div v-if="store.status !== 'replaying'">
      <p v-if="store.gameRecords.length === 0" class="text-gray-500 text-sm">暂无棋谱记录</p>
      <div v-else class="space-y-2 max-h-64 overflow-y-auto">
        <div
          v-for="record in store.gameRecords"
          :key="record.id"
          class="bg-gray-800 rounded-lg p-3 cursor-pointer hover:bg-gray-700 transition-colors border border-gray-700"
          @click="store.startReplay(record)"
        >
          <div class="flex justify-between items-center">
            <span class="text-sm text-gray-300">{{ record.createdAt }}</span>
            <span
              class="text-xs px-2 py-0.5 rounded-full"
              :class="record.winner === 1 ? 'bg-gray-700 text-gray-200' : record.winner === 2 ? 'bg-white text-black' : 'bg-yellow-600 text-white'"
            >
              {{ record.winner === 1 ? '黑棋胜' : record.winner === 2 ? '白棋胜' : '平局' }}
            </span>
          </div>
          <div class="text-xs text-gray-500 mt-1">共 {{ record.moves.length }} 手</div>
        </div>
      </div>
    </div>

    <div v-else>
      <div class="text-center mb-2">
        <span class="text-gray-400 text-sm">第 {{ store.replayIndex }} / {{ store.replayMoves.length }} 手</span>
      </div>

      <div
        ref="trackRef"
        class="relative w-full h-3 bg-gray-800 rounded-full cursor-pointer mb-1 group"
        @mousedown="onTrackMouseDown"
      >
        <div
          class="absolute top-0 left-0 h-full bg-green-500/40 rounded-full pointer-events-none"
          :style="{ width: progressPercent }"
        />
        <div
          class="absolute top-1/2 -translate-y-1/2 w-4 h-4 bg-green-500 rounded-full shadow-lg border-2 border-green-300 pointer-events-none transition-[left] duration-75"
          :style="{ left: `calc(${progressPercent} - 8px)` }"
        />
      </div>

      <div class="flex justify-between text-xs text-gray-600 mb-4">
        <span>0</span>
        <span>{{ store.replayMoves.length }}</span>
      </div>

      <div
        v-if="store.replayCurrentResult"
        class="text-center text-sm mb-3 py-1.5 px-3 rounded-lg"
        :class="resultClass"
      >
        {{ store.replayCurrentResult }}
      </div>

      <div class="flex items-center justify-center gap-2 mb-4">
        <button @click="store.replayGoToStart()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="回到开始">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 19l-7-7 7-7m8 14l-7-7 7-7"/></svg>
        </button>
        <button @click="store.replayStepBack()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="上一步">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"/></svg>
        </button>
        <button @click="store.toggleReplayPlay()" class="p-3 bg-green-600 rounded-lg hover:bg-green-500 text-white transition-colors" :title="store.isReplayPlaying ? '暂停' : '播放'">
          <svg v-if="!store.isReplayPlaying" class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24"><path d="M8 5v14l11-7z"/></svg>
          <svg v-else class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24"><path d="M6 4h4v16H6zM14 4h4v16h-4z"/></svg>
        </button>
        <button @click="store.replayStepForward()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="下一步">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"/></svg>
        </button>
        <button @click="store.replayGoToEnd()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="跳到结尾">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 5l7 7-7 7M5 5l7 7-7 7"/></svg>
        </button>
      </div>

      <div class="flex items-center justify-center gap-2 mb-4">
        <span class="text-xs text-gray-500">速度:</span>
        <button
          v-for="speed in speeds"
          :key="speed.value"
          @click="store.setReplaySpeed(speed.value)"
          class="px-2 py-1 text-xs rounded transition-colors"
          :class="store.replaySpeed === speed.value ? 'bg-green-600 text-white' : 'bg-gray-800 text-gray-400 hover:bg-gray-700'"
        >
          {{ speed.label }}
        </button>
      </div>

      <button
        @click="store.stopReplay()"
        class="w-full py-2 bg-red-600/20 border border-red-600/50 text-red-400 rounded-lg hover:bg-red-600/30 transition-colors text-sm"
      >
        退出回放
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onUnmounted } from 'vue';
import { useGameStore } from '../store/game';

const store = useGameStore();
const trackRef = ref<HTMLElement | null>(null);

const speeds = [
  { label: '慢', value: 2000 },
  { label: '中', value: 1000 },
  { label: '快', value: 500 },
  { label: '极快', value: 200 },
];

const progressPercent = computed(() => {
  if (store.replayMoves.length === 0) return '0%';
  return `${(store.replayIndex / store.replayMoves.length) * 100}%`;
});

const resultClass = computed(() => {
  const text = store.replayCurrentResult;
  if (!text) return '';
  if (text.includes('黑棋胜')) return 'bg-gray-700/60 text-gray-200';
  if (text.includes('白棋胜')) return 'bg-white/20 text-white';
  if (text.includes('平局')) return 'bg-yellow-600/30 text-yellow-300';
  return 'bg-gray-800 text-gray-400';
});

function getIndexFromEvent(e: MouseEvent): number {
  const track = trackRef.value;
  if (!track) return store.replayIndex;
  const rect = track.getBoundingClientRect();
  const x = e.clientX - rect.left;
  const ratio = Math.max(0, Math.min(1, x / rect.width));
  return Math.round(ratio * store.replayMoves.length);
}

let isDragging = false;

function onTrackMouseDown(e: MouseEvent) {
  isDragging = true;
  const idx = getIndexFromEvent(e);
  store.replayGoToMove(idx);
  window.addEventListener('mousemove', onMouseMove);
  window.addEventListener('mouseup', onMouseUp);
}

function onMouseMove(e: MouseEvent) {
  if (!isDragging) return;
  const idx = getIndexFromEvent(e);
  store.replayGoToMove(idx);
}

function onMouseUp() {
  isDragging = false;
  window.removeEventListener('mousemove', onMouseMove);
  window.removeEventListener('mouseup', onMouseUp);
}

onUnmounted(() => {
  window.removeEventListener('mousemove', onMouseMove);
  window.removeEventListener('mouseup', onMouseUp);
});
</script>
