<script setup lang="ts">
import type {
    DropdownItemProps,
    DropdownProps,
    DropdownEmits,
    DropdownContext,
    DropdownInstance
} from "./types"

import {useDisabledStyle} from "@zznikki-element/hooks"
import type { TooltipInstance } from "../Tooltip/types";
import { type ButtonInstance,ErButton,ErButtonGroup } from "../Button/index.ts";
import {ref,computed,provide} from 'vue'
import {omit,isNil} from 'lodash-es'
import DropdownItem from "./DropdownItem.vue"
import ErTooltip from "../Tooltip/Tooltip.vue"
import {DROPDOWN_CTX_KEY} from './constants'

defineOptions({
    name: "ErDropdown",
    inheritAttrs:false
})

const props = withDefaults(defineProps<DropdownProps>(), {
    hideOnClick:true,
    items:()=>[] as DropdownItemProps[]
})
const emits=defineEmits<DropdownEmits>()
const slots=defineSlots()

const tooltipRef=ref<TooltipInstance>()
const triggerRef=ref<ButtonInstance>()
const virtualRef = computed(()=>triggerRef.value?.ref ?? void 0)

const tooltipProps = computed(()=>
    omit(props,["items","hideAfterClick","size","type","splitButton"])
)

function handleItemClick(e:DropdownItemProps){
    props.hideOnClick && tooltipRef.value?.hide()
    !isNil(e.command) && emits('command',e.command)
}
!TEST && useDisabledStyle();
provide<DropdownContext>(DROPDOWN_CTX_KEY, {
  handleItemClick,
  size: computed(() => props.size),
});

defineExpose<DropdownInstance>({
    open:() => tooltipRef.value?.show(),
    close:() => tooltipRef.value?.hide()
})
</script>

<template>
 <div class="er-dropdown" :class="{ 'is-disabled': props.disabled }">
    <er-tooltip
      ref="tooltipRef"
      v-bind="tooltipProps"
      :virtual-triggering="splitButton"
      :virtual-ref="virtualRef?.value"
      @visible-change="$emit('visible-change', $event)"
    >
      <er-button-group
        v-if="splitButton"
        :type="type"
        :size="size"
        :disabled="disabled"
      >
        <er-button @click="$emit('click', $event)">
          <slot name="default"></slot>
        </er-button>
        <er-button ref="triggerRef" icon="angle-down" />
      </er-button-group>
      <slot name="default" v-else></slot>

      <template #content>
        <div class="er-dropdown__menu">
          <slot name="dropdown">
            <template v-for="item in items" :key="item.command">
              <dropdown-item v-bind="item" />
            </template>
          </slot>
        </div>
      </template>
    </er-tooltip>
  </div>
</template>

<style >
@import './style.css';
:deep(.er-button-group) {
  & > :last-child {
    padding: 5px 7px;
  }
}
</style>
