<script setup lang="ts">
import { ref, onMounted, reactive, watch } from 'vue';
import PromotionDialog from './PromotionDialog.vue';
import { BoardApi } from '@/classes/BoardApi';
import type { BoardState, Props, Emits } from '@/typings/Chessboard';
import type { BoardConfig } from '@/typings/BoardConfig';
import { deepCopy, deepDiffConfig } from '@/helper/Board';

const props = withDefaults(defineProps<Props>(), {
  boardConfig: () => ({}),
  reactiveConfig: false,
});
const emit = defineEmits<Emits>();

const boardElement = ref<HTMLElement | null>(null);
const boardState: BoardState = reactive({
  showThreats: false,
  promotionDialogState: { isEnabled: false },
  historyViewerState: { isEnabled: false },
});

onMounted(() => {
  if (boardElement.value == null) {
    throw new Error('vue3-chessboard: Failed to mount board.');
  }

  const boardAPI = new BoardApi(boardElement.value, boardState, props, emit);
  emit('boardCreated', boardAPI);

  if (props.reactiveConfig) {
    let oldConfig: BoardConfig = deepCopy(props.boardConfig);
    watch(reactive(props.boardConfig), (newConfig: BoardConfig) => {
      boardAPI.setConfig(deepDiffConfig(oldConfig, newConfig));
      oldConfig = deepCopy(newConfig);
    });
  }
});
</script>

<template>
  <section
    class="main-wrap"
    :class="{
      disabledBoard: boardState.promotionDialogState.isEnabled,
      viewingHistory: boardState.historyViewerState.isEnabled,
    }"
  >
    <div class="main-board">
      <div class="dialog-container">
        <PromotionDialog
          v-if="boardState.promotionDialogState.isEnabled"
          :state="boardState.promotionDialogState"
          @promotion-selected="
            boardState.promotionDialogState = { isEnabled: false }
          "
        />
      </div>
      <div ref="boardElement"></div>
    </div>
  </section>
</template>
