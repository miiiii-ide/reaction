<template>
  <div class="section" @click.self="closeReactionPicker()">
    <span
      type="is-light"
      @click="showReactionPicker()"
      rounded
      size="is-small"
      @keyup.esc="closeReactionPicker()"
      class="add-reaction"
    >
      <div class="reaction-wrapper">
        <div v-for="(item, index) in state.reaction.items" :key="index" class="reaction-button">
          <span>{{item.unified}}</span>
          <span>{{item.count}}</span>
        </div>
      </div>
      <b-icon icon="emoticon-happy-outline" />
      <b-icon icon="plus" size="is-small" />
    </span>
    <div v-if="!isClosed">
      <Picker
        title="pick reaction"
        emoji="grinning"
        set="twitter"
        color="pink"
        @select="selectReaction"
        :sheetSize="state.sheetSize"
        :include="category"
      />
    </div>
  </div>
</template>
<script lang="ts">
import {
  defineComponent,
  ref,
  reactive,
  watchEffect,
} from '@vue/composition-api'
import { Picker } from 'emoji-mart-vue'
import { IReactionItem } from '@/models/Reaction'

export default defineComponent({
  name: 'Reaction',
  components: {
    Picker,
  },
  setup() {
    const state = reactive({
      sheetSize: 32 as number,
      selectedReactions: [],
      reaction: {
        hasReaction: false as boolean,
        items: [
          {
            reactionId: '' as string,
            count: 0 as number,
            unified: '' as string,
          },
        ] as Array<IReactionItem>,
      },
    })

    const reactionItems: IReactionItem[] = [
      {
        reactionId: '1f606',
        count: 2,
        unified: '0x1f606',
      },
      {
        reactionId: '1f34e',
        count: 3,
        unified: '0x1f34e',
      },
    ]

    const reactions = ref<IReactionItem[] | undefined>(reactionItems)

    const category = ['search', 'recent', 'people', 'foods']

    const isClosed = ref<boolean>(true)

    function showReactionPicker() {
      isClosed.value = !isClosed.value
    }

    function closeReactionPicker() {
      if (!isClosed.value) {
        isClosed.value = true
      }
    }

    watchEffect(async () => {
      reactions.value = await reactionItems

      if (!reactions.value) {
        return
      }

      state.reaction.items = []
      state.reaction.hasReaction = false

      reactions.value.forEach((r: IReactionItem, index: number) => {
        let splitItems: string[] = r.reactionId.split('-')
        splitItems.forEach((item: string, index: number) => {
          splitItems[index] = `0x${item}`
        })

        let group = {
          reactionId: r.reactionId,
          count: 0,
          unified: '',
        }
        // @ts-ignore
        group.unified = String.fromCodePoint(...splitItems)
        group.count = r.count

        state.reaction.items.push(group)
      })
    })

    function selectReaction(item: any) {
      if (!reactions.value) {
        return
      }

      const target = reactions.value.findIndex(
        (r) => r.reactionId === item.unified
      )
      if (target >= 0) {
        return
      }

      state.reaction.hasReaction = false

      let splitItems: string[] = item.unified.split('-')
      splitItems.forEach((i: string, index: number) => {
        splitItems[index] = `0x${i}`
      })

      let group = {
        reactionId: item.unified,
        count: 0,
        unified: '',
      }

      // @ts-ignore
      group.unified = String.fromCodePoint(...splitItems)
      group.count = 1

      state.reaction.items.push(group)

      isClosed.value = true
    }

    return {
      state,
      category,
      isClosed,
      showReactionPicker,
      closeReactionPicker,
      selectReaction,
    }
  },
})
</script>

<style lang="scss">
.add-reaction {
  cursor: pointer;
}
.reaction-button {
  border: 1px blue solid;
  width: 3rem;
  border-radius: 20px;
  padding: 0 0.5rem;
  margin-right: 0.5rem;
}
.reaction-wrapper {
  display: flex;
  margin-bottom: 2rem;
}
</style>