<script setup lang="ts">
import { computed, useAttrs, type PropType, ref, watch, nextTick } from "vue";

import ODropdown from "../dropdown/Dropdown.vue";
import ODropdownItem from "../dropdown/DropdownItem.vue";
import OInput from "../input/Input.vue";

import {
    useEventListener,
    useInputHandler,
    usePropBinding,
} from "@/composables";
import { isMobileAgent } from "@/utils/helpers";
import { isClient } from "@/utils/ssr";
import type { ComponentClass, PropBind } from "@/types";

/**
 * This is a internal used component.
 * Used by Datepicker and Timepicker.
 */
defineOptions({
    name: "OPickerWrapper",
});

const props = defineProps({
    /** parent picker component props  */
    pickerProps: { type: Object, required: true },
    /** data-oruga attribute value */
    dataOruga: { type: String, required: true },
    /** the input value */
    value: { type: [Date, Array], default: undefined },
    /** the active state of the dropdown */
    active: { type: Boolean, default: false },
    formattedValue: { type: String, default: undefined },
    nativeType: { type: String, required: true },
    nativeStep: { type: String, default: undefined },
    nativeValue: { type: [String, Number], default: undefined },
    nativeMin: { type: [String, Number], default: undefined },
    nativeMax: { type: [String, Number], default: undefined },
    stayOpen: { type: Boolean, default: false },
    dropdownClass: { type: String, required: true },
    rootClasses: {
        type: [String, Object] as PropType<PropBind>,
        required: true,
    },
    boxClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        required: true,
    },
});

const emits = defineEmits<{
    /**
     * active prop two-way binding
     * @param value {boolean} updated active prop
     */
    (e: "update:active", value: boolean): void;
    /** on value change event */
    (e: "change", value: string): void;
    /** on natvie value change event */
    (e: "native-change", value: string): void;
    /** on input focus event */
    (e: "focus", evt: Event): void;
    /** on input blur event */
    (e: "blur", evt: Event): void;
    /** on input invalid event */
    (e: "invalid", evt: Event): void;
    /** on icon click event */
    (e: "icon-click", evt: Event): void;
    /** on icon right click event */
    (e: "icon-right-click", evt: Event): void;
}>();

/** the computed picker contains all chared props from the datepicker and the timepicker  */
const picker = computed<any>(() => props.pickerProps);

const isMobileNative = computed(
    () => picker.value.mobileNative && isMobileAgent.any(),
);

const dropdownRef = ref<InstanceType<typeof ODropdown>>();
const inputRef = ref<InstanceType<typeof OInput>>();
const nativeInputRef = ref<InstanceType<typeof OInput>>();

const elementRef = computed(() =>
    isMobileNative.value ? nativeInputRef.value : inputRef.value,
);

// use form input functionality for native input
const {
    checkHtml5Validity,
    setFocus,
    doClick,
    onBlur,
    onFocus,
    onInvalid,
    isValid,
    isFocused,
} = useInputHandler(elementRef, emits, picker.value);

/**
 * Show input as text for placeholder,
 * when placeholder and native value is given and input is not focused.
 */
const computedNativeType = computed(() =>
    !picker.value.placeholder || props.nativeValue || isFocused.value
        ? props.nativeType
        : "text",
);

/**
 * When v-model is changed:
 *  1. Update internal value.
 *  2. If it's invalid, validate again.
 */
watch(
    () => props.value,
    () => {
        // toggle picker if not stay open
        if (!props.stayOpen) togglePicker(false);
        // check validation if native
        if (isMobileNative.value && !isValid.value) checkHtml5Validity();
    },
);

const isActive = usePropBinding<boolean>("active", props, emits, {
    passive: true,
});

watch(isActive, onActiveChange);

const ariaRole = computed(() => (!picker.value.inline ? "dialog" : undefined));

const triggers = computed(() => (picker.value.openOnFocus ? ["click"] : []));

if (isClient) useEventListener("keyup", onKeyPress);

/** Keypress event that is bound to the document. */
function onKeyPress(event: KeyboardEvent): void {
    if (isActive.value && (event.key === "Escape" || event.key === "Esc"))
        togglePicker(false);
}

// --- EVENT HANDLER ---

/** Toggle picker */
function togglePicker(active: boolean): void {
    if (isMobileNative.value) {
        setFocus(); // focus the underlaying input element
        doClick(); // click to open the underlaying input element
    } else if (dropdownRef.value) {
        if (active || picker.value.closeOnClick)
            nextTick(() => (isActive.value = active));
    }
}

/** Avoid dropdown toggle when is already visible */
function onInputClick(event): void {
    if (isActive.value) event.stopPropagation();
}

/** Emit 'blur' event on dropdown is not active (closed) */
function onActiveChange(value: boolean): void {
    if (value) onFocus();
    else if (!value) onBlur();
}

// --- Computed Component Classes ---

const attrs = useAttrs();
const inputBind = computed(() => ({
    ...attrs,
    ...picker.value.inputClasses,
}));

const dropdownBind = computed(() => ({
    "root-class": props.dropdownClass,
    ...picker.value.dropdownClasses,
}));
</script>

<template>
    <div :data-oruga="dataOruga" :class="rootClasses">
        <o-dropdown
            v-if="!isMobileNative || picker.inline"
            ref="dropdownRef"
            v-bind="dropdownBind"
            v-model:active="isActive"
            :position="picker.position"
            :disabled="picker.disabled"
            :inline="picker.inline"
            :mobile-modal="picker.mobileModal"
            :trap-focus="picker.trapFocus"
            :aria-role="ariaRole"
            :aria-modal="!picker.inline"
            :trigger-tabindex="-1"
            :teleport="picker.teleport"
            :triggers="triggers">
            <template v-if="!picker.inline" #trigger>
                <slot name="trigger">
                    <o-input
                        ref="inputRef"
                        v-bind="inputBind"
                        autocomplete="off"
                        :model-value="formattedValue"
                        :placeholder="picker.placeholder"
                        :size="picker.size"
                        :icon-pack="picker.iconPack"
                        :icon="picker.icon"
                        :icon-right="picker.iconRight"
                        :icon-right-clickable="picker.iconRightClickable"
                        :expanded="picker.expanded"
                        :rounded="picker.rounded"
                        :disabled="picker.disabled"
                        :readonly="picker.readonly"
                        :use-html5-validation="false"
                        @click="onInputClick"
                        @keyup.enter="togglePicker(true)"
                        @change="$emit('change', $event.target.value)"
                        @focus="onFocus"
                        @blur="onBlur"
                        @icon-click="$emit('icon-click', $event)"
                        @icon-right-click="$emit('icon-right-click', $event)" />
                </slot>
            </template>
            <o-dropdown-item
                override
                tag="div"
                :item-class="boxClass"
                :disabled="picker.disabled"
                :clickable="false">
                <slot />
            </o-dropdown-item>
        </o-dropdown>

        <!-- Native Picker -->
        <o-input
            v-else
            ref="nativeInputRef"
            v-bind="inputBind"
            :type="computedNativeType"
            autocomplete="off"
            :model-value="nativeValue"
            :min="nativeMin"
            :max="nativeMax"
            :step="nativeStep"
            :placeholder="picker.placeholder"
            :size="picker.size"
            :icon-pack="picker.iconPack"
            :icon="picker.icon"
            :icon-right="picker.iconRight"
            :icon-right-clickable="picker.iconRightClickable"
            :rounded="picker.rounded"
            :disabled="picker.disabled"
            :readonly="false"
            :use-html5-validation="false"
            @change="$emit('native-change', $event.target.value)"
            @focus="onFocus"
            @blur="onBlur"
            @invalid="onInvalid"
            @icon-click="$emit('icon-click', $event)"
            @icon-right-click="$emit('icon-right-click', $event)" />
    </div>
</template>
