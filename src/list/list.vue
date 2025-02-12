<template>
  <div ref="root" :class="name" @scroll="handleScroll">
    <t-node :content="headerContent" />
    <slot />
    <div @click.stop="onLoadMore">
      <t-loading
        v-if="typeof asyncLoading === 'string' && ['loading', 'loadingMore'].includes(asyncLoading)"
        :loading="asyncLoading === 'loading'"
        :text="typeof asyncLoading === 'string' ? LOADING_TEXT_MAP[asyncLoading] : ''"
        :class="`${name}__loading`"
      />
    </div>
    <t-node :content="footerContent" />
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed, getCurrentInstance, SetupContext } from 'vue';
import { useElementBounding, useWindowSize, useEventListener } from '@vueuse/core';
import TLoading from '../loading';
import config from '../config';
import ListProps from './props';
import { renderTNode, TNode, useScrollParent, useEmitEvent } from '../shared';

const { prefix } = config;
const name = `${prefix}-list`;
const LOADING_TEXT_MAP = {
  loading: '加载中...',
  'loading-more': '点击加载更多',
};

export default defineComponent({
  name,
  components: {
    TLoading,
    TNode,
  },
  props: ListProps,
  emits: ['load-more', 'scroll'],
  setup(props, context: SetupContext) {
    const emitEvent = useEmitEvent(props, context.emit);
    const root = ref<HTMLElement>();
    const empty = ref<HTMLElement>();
    const scrollParent = useScrollParent(root);
    const { height } = useWindowSize();
    const internalInstance = getCurrentInstance();

    const headerContent = computed(() => renderTNode(internalInstance, 'header'));
    const footerContent = computed(() => renderTNode(internalInstance, 'footer'));

    const onLoadMore = (e: MouseEvent) => {
      if (props.asyncLoading === 'loading-more') {
        emitEvent('load-more');
      }
    };

    const handleScroll = (e: WheelEvent | Event) => {
      const { bottom } = useElementBounding(root);
      emitEvent('scroll', bottom.value - height.value);
    };

    useEventListener(scrollParent, 'scroll', handleScroll);

    return {
      name,
      root,
      empty,
      onLoadMore,
      handleScroll,
      headerContent,
      footerContent,
      LOADING_TEXT_MAP,
    };
  },
});
</script>
