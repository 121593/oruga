<script setup lang="ts">
import { computed, ref, watch, type PropType } from "vue";

import ODatepicker from "../datepicker/Datepicker.vue";
import OTimepicker from "../timepicker/Timepicker.vue";
import OInput from "../input/Input.vue";

import { baseComponentProps } from "@/utils/SharedProps";
import { getOption } from "@/utils/config";
import { isMobileAgent } from "@/utils/helpers";
import {
    useComputedClass,
    useInputHandler,
    usePropBinding,
} from "@/composables";

import { matchWithGroups } from "../datepicker/utils";
import type { DatepickerProps } from "../datepicker/useDatepickerShare";
import type { TimepickerProps } from "../timepicker/useTimepickerShare";

import type { ComponentClass } from "@/types";

/**
 * An input with a simple dropdown/modal for selecting a date and time, uses native datetimepicker for mobile
 * @displayName Datetimepicker
 * @style _datetimepicker.scss
 */
defineOptions({
    isOruga: true,
    name: "ODatetimepicker",
    configField: "datetimepicker",
    inheritAttrs: false,
});

const AM = "AM";
const PM = "PM";
const HOUR_FORMAT_24 = "24";

const props = defineProps({
    // add global shared props (will not be displayed in the docs)
    ...baseComponentProps,
    /** @model */
    modelValue: { type: Date, default: undefined },
    /** The active state of the dropdown, use v-model:active to make it two-way binding. */
    active: { type: Boolean, default: false },
    /** Define props for the underlying datepicker component */
    datepicker: {
        type: Object as PropType<DatepickerProps>,
        default: undefined,
    },
    /** Define props for the underlying timepicker component */
    timepicker: {
        type: Object as PropType<TimepickerProps>,
        default: undefined,
    },
    /** Min date to select */
    minDatetime: { type: Date, default: undefined },
    /** Max date to select */
    maxDatetime: { type: Date, default: undefined },
    /**
     * Size of the input control
     * @values small, medium, large
     */
    size: {
        type: String,
        default: () => getOption("datetimepicker.size"),
    } /** Makes input full width when inside a grouped or addon field */,
    expanded: { type: Boolean, default: false },
    /** Makes the input rounded */
    rounded: { type: Boolean, default: false },
    /** Input placeholder */
    placeholder: { type: String, default: undefined },
    /** Same as native input readonly */
    readonly: { type: Boolean, default: false },
    /** Same as native disabled */
    disabled: { type: Boolean, default: false },
    /** Display datetimepicker inline */
    inline: { type: Boolean, default: false },
    /** Open dropdown on focus */
    openOnFocus: {
        type: Boolean,
        default: () => getOption("datetimepicker.openOnFocus", true),
    },
    /** Date format locale */
    locale: {
        type: String,
        default: () => getOption("locale"),
    },
    /** Custom function to format a date into a string */
    datetimeFormatter: {
        type: Function as PropType<(date: Date) => string>,
        default: (
            date: Date | Date[],
            defaultFunction: (date: Date | Date[]) => string,
        ) => getOption("datetimepicker.dateFormatter", defaultFunction)(date),
    },
    /** Custom function to parse a string into a date */
    datetimeParser: {
        type: Function as PropType<(date: string) => Date>,
        default: (date: string, defaultFunction: (date: string) => Date) =>
            getOption("datetimepicker.dateParser", defaultFunction)(date),
    },
    /** Date creator function, default is `new Date()` */
    datetimeCreator: {
        type: Function as PropType<(date: Date) => Date>,
        default: (date: Date) =>
            getOption(
                "datetimepicker.datetimeCreator",
                (d: Date) => new Date(d),
            )(date),
    },
    /** Dropdown position */
    position: { type: String, default: undefined },
    /** Enable mobile native input if mobile agent */
    mobileNative: {
        type: Boolean,
        default: () => getOption("datetimepicker.mobileNative", true),
    },
    /**
     * Icon pack to use
     * @values mdi, fa, fas and any other custom icon pack
     */
    iconPack: {
        type: String,
        default: () => getOption("datetimepicker.iconPack", undefined),
    },
    /** Icon to be shown */
    icon: {
        type: String,
        default: () => getOption("datetimepicker.icon", undefined),
    },
    /** Icon to be added on the right side */
    iconRight: {
        type: String,
        default: () => getOption("datetimepicker.iconRight", undefined),
    },
    /** Make the icon right clickable */
    iconRightClickable: { type: Boolean, default: false },
    /**
     * Append the component to another part of the DOM.
     * Set `true` to append the component to the body.
     * In addition, any CSS selector string or an actual DOM node can be used.
     */
    teleport: {
        type: [Boolean, String, Object],
        default: () => getOption("datetimepicker.teleport", false),
    },
    /** Enable html 5 native validation */
    useHtml5Validation: {
        type: Boolean,
        default: () => getOption("useHtml5Validation", true),
    },
    /** The message which is shown when a validation error occurs */
    validationMessage: { type: String, default: undefined },
    // class props (will not be displayed in the docs)
    datepickerWrapperClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    timepickerWrapperClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
});

const emits = defineEmits<{
    /**
     * modelValue prop two-way binding
     * @param value {Date | Date[]} updated modelValue prop
     */
    (e: "update:modelValue", value: Date | Date[]): void;
    /**
     * active prop two-way binding
     * @param value {boolean} updated active prop
     */
    (e: "update:active", value: boolean): void;
    /**
     * on range start is selected event
     * @param value {Date} range start date
     */
    (e: "range-start", value: Date): void;
    /**
     * on range end is selected event
     * @param value {Date} range end date
     */
    (e: "range-end", value: Date): void;
    /**
     * on month change event
     * @param value {number} month number
     */
    (e: "change-month", value: number): void;
    /**
     * on year change event
     * @param value {number} year number
     */
    (e: "change-year", value: number): void;
    /**
     * on input focus event
     * @param event {Event} native event
     */
    (e: "focus", event: Event): void;
    /**
     * on input blur event
     * @param event {Event} native event
     */
    (e: "blur", event: Event): void;
    /**
     * on input invalid event
     * @param event {Event} native event
     */
    (e: "invalid", event: Event): void;
    /**
     * on icon click event
     * @param event {Event} native event
     */
    (e: "icon-click", event: Event): void;
    /**
     * on icon right click event
     * @param event {Event} native event
     */
    (e: "icon-right-click", event: Event): void;
}>();

const datepickerRef = ref<InstanceType<typeof ODatepicker>>();
const timepickerRef = ref<InstanceType<typeof OTimepicker>>();
const nativeInputRef = ref<InstanceType<typeof OInput>>();

const elementRef = computed(() =>
    isMobileNative.value ? nativeInputRef.value : datepickerRef.value,
);

// use form input functionality for native input
const { onBlur, onFocus, onInvalid } = useInputHandler(
    elementRef,
    emits,
    props,
);

const isMobileNative = computed(
    () => props.mobileNative && isMobileAgent.any(),
);

watch([() => isMobileNative.value, () => props.inline], () => {
    // $refs attached, it's time to refresh datepicker (input)
    if (datepickerRef.value) datepickerRef.value.$forceUpdate();
});

/** Dropdown active state */
const isActive = usePropBinding<boolean>("active", props, emits);

const vmodel = computed({
    get() {
        return props.modelValue;
    },
    set(value) {
        if (!value) {
            emits("update:modelValue", value);
            return;
        }
        let date = new Date(value.getTime());
        if (!props.modelValue) {
            date = props.datetimeCreator(value);
        } else {
            // restore time part
            if (
                (value.getDate() !== props.modelValue.getDate() ||
                    value.getMonth() !== props.modelValue.getMonth() ||
                    value.getFullYear() !== props.modelValue.getFullYear()) &&
                value.getHours() === 0 &&
                value.getMinutes() === 0 &&
                value.getSeconds() === 0
            ) {
                date.setHours(
                    props.modelValue.getHours(),
                    props.modelValue.getMinutes(),
                    props.modelValue.getSeconds(),
                    0,
                );
            }
        }
        // check min and max range
        if (props.minDatetime && date < props.minDatetime) {
            date = props.minDatetime;
        } else if (props.maxDatetime && date > props.maxDatetime) {
            date = props.maxDatetime;
        }
        emits("update:modelValue", new Date(date.getTime()));
    },
});

const minDate = computed(() => {
    if (!props.minDatetime)
        return props.datepicker ? props.datepicker.minDate : null;

    return new Date(
        props.minDatetime.getFullYear(),
        props.minDatetime.getMonth(),
        props.minDatetime.getDate(),
        0,
        0,
        0,
        0,
    );
});

const maxDate = computed(() => {
    if (!props.maxDatetime)
        return props.datepicker ? props.datepicker.maxDate : null;

    return new Date(
        props.maxDatetime.getFullYear(),
        props.maxDatetime.getMonth(),
        props.maxDatetime.getDate(),
        0,
        0,
        0,
        0,
    );
});

// Only enable min/max time if local (not necessarily UTC) date portion matches
const minTime = computed(() => {
    if (
        !props.minDatetime ||
        vmodel.value === null ||
        typeof vmodel.value === "undefined" ||
        vmodel.value.getFullYear() != props.minDatetime.getFullYear() ||
        vmodel.value.getMonth() != props.minDatetime.getMonth() ||
        vmodel.value.getDate() != props.minDatetime.getDate()
    ) {
        return props.timepicker ? props.timepicker.minTime : null;
    }
    return props.minDatetime;
});

const maxTime = computed(() => {
    if (
        !props.maxDatetime ||
        vmodel.value === null ||
        typeof vmodel.value === "undefined" ||
        vmodel.value.getFullYear() != props.maxDatetime.getFullYear() ||
        vmodel.value.getMonth() != props.maxDatetime.getMonth() ||
        vmodel.value.getDate() != props.maxDatetime.getDate()
    ) {
        return props.timepicker ? props.timepicker.maxTime : null;
    }
    return props.maxDatetime;
});

const datepickerSize = computed(() =>
    props.datepicker?.size ? props.datepicker.size : props.size,
);

const timepickerSize = computed(() =>
    props.timepicker?.size ? props.timepicker.size : props.size,
);

const timepickerDisabled = computed(() =>
    props.timepicker?.disabled ? props.timepicker.disabled : props.disabled,
);

function formatNative(value: Date): string {
    const date = new Date(value);
    if (value && !isNaN(date.getTime())) {
        const year = date.getFullYear();
        const month = date.getMonth() + 1;
        const day = date.getDate();
        const hours = date.getHours();
        const minutes = date.getMinutes();
        const seconds = date.getSeconds();
        return (
            year +
            "-" +
            ((month < 10 ? "0" : "") + month) +
            "-" +
            ((day < 10 ? "0" : "") + day) +
            "T" +
            ((hours < 10 ? "0" : "") + hours) +
            ":" +
            ((minutes < 10 ? "0" : "") + minutes) +
            ":" +
            ((seconds < 10 ? "0" : "") + seconds)
        );
    }
    return "";
}

// --- Time Format Feature ---

const enableSeconds = computed(() =>
    timepickerRef.value?.enableSeconds
        ? timepickerRef.value.enableSeconds
        : false,
);

const localeOptions = computed(
    () =>
        new Intl.DateTimeFormat(props.locale, {
            year: "numeric",
            month: "numeric",
            day: "numeric",
            hour: "numeric",
            minute: "numeric",
            second: enableSeconds.value ? "numeric" : undefined,
        }).resolvedOptions() as Intl.DateTimeFormatOptions,
);

const isHourFormat24 = computed(
    () =>
        props.timepicker?.hourFormat === HOUR_FORMAT_24 ||
        !localeOptions.value.hour12,
);

const dtf = computed(
    () =>
        new Intl.DateTimeFormat(props.locale, {
            year: localeOptions.value.year || "numeric",
            month: localeOptions.value.month || "numeric",
            day: localeOptions.value.day || "numeric",
            hour: localeOptions.value.hour || "numeric",
            minute: localeOptions.value.minute || "numeric",
            second: enableSeconds.value
                ? localeOptions.value.second || "numeric"
                : undefined,
            hourCycle: !isHourFormat24.value ? "h12" : "h23",
        }),
);

const amString = computed(() => {
    if (
        dtf.value.formatToParts &&
        typeof dtf.value.formatToParts === "function"
    ) {
        const d = props.datetimeCreator(new Date());
        d.setHours(10);
        const dayPeriod = dtf.value
            .formatToParts(d)
            .find((part) => part.type === "dayPeriod");
        if (dayPeriod) return dayPeriod.value;
    }
    return AM;
});

const pmString = computed(() => {
    if (
        dtf.value.formatToParts &&
        typeof dtf.value.formatToParts === "function"
    ) {
        const d = props.datetimeCreator(new Date());
        d.setHours(20);
        const dayPeriod = dtf.value
            .formatToParts(d)
            .find((part) => part.type === "dayPeriod");
        if (dayPeriod) {
            return dayPeriod.value;
        }
    }
    return PM;
});

function defaultDatetimeParser(value: string): Date {
    function defaultParser(date: string): Date {
        if (
            dtf.value.formatToParts &&
            typeof dtf.value.formatToParts === "function"
        ) {
            const dayPeriods = [AM, PM, AM.toLowerCase(), PM.toLowerCase()];
            if (timepickerRef.value) {
                dayPeriods.push(amString.value);
                dayPeriods.push(pmString.value);
            }
            const parts = this.dtf.formatToParts(new Date());
            const formatRegex = parts
                .map((part, idx) => {
                    if (part.type === "literal") {
                        if (
                            idx + 1 < parts.length &&
                            parts[idx + 1].type === "hour"
                        ) {
                            return `[^\\d]+`;
                        }
                        return part.value.replace(/ /g, "\\s?");
                    } else if (part.type === "dayPeriod") {
                        return `((?!=<${part.type}>)(${dayPeriods.join(
                            "|",
                        )})?)`;
                    }
                    return `((?!=<${part.type}>)\\d+)`;
                })
                .join("");
            const datetimeGroups = matchWithGroups(formatRegex, date);

            // We do a simple validation for the group.
            // If it is not valid, it will fallback to Date.parse below
            if (
                datetimeGroups.year &&
                datetimeGroups.year.length === 4 &&
                datetimeGroups.month &&
                datetimeGroups.month <= 12 &&
                datetimeGroups.day &&
                datetimeGroups.day <= 31 &&
                datetimeGroups.hour &&
                datetimeGroups.hour >= 0 &&
                datetimeGroups.hour < 24 &&
                datetimeGroups.minute &&
                datetimeGroups.minute >= 0 &&
                datetimeGroups.minute <= 59
            ) {
                const d = new Date(
                    datetimeGroups.year,
                    datetimeGroups.month - 1,
                    datetimeGroups.day,
                    datetimeGroups.hour,
                    datetimeGroups.minute,
                    datetimeGroups.second || 0,
                );
                return d;
            }
        }

        return new Date(Date.parse(date));
    }
    const date = (props.datetimeParser as any)(value, defaultParser);
    return date;
}

function defaultDatetimeFormatter(date: Date): string {
    return (props.datetimeFormatter as any)(date, (date: Date) =>
        date ? dtf.value.format(date) : "",
    );
}

// --- Event Handler ---

/** Parse date from string */
function onChangeNativePicker(event: Event): void {
    const date = (event.target as HTMLInputElement).value;
    const s = date ? date.split(/\D/) : [];
    if (s.length >= 5) {
        const year = parseInt(s[0], 10);
        const month = parseInt(s[1], 10) - 1;
        const day = parseInt(s[2], 10);
        const hours = parseInt(s[3], 10);
        const minutes = parseInt(s[4], 10);
        // Seconds are omitted intentionally; they are unsupported by input
        // type=datetime-local and cause the control to fail native validation
        vmodel.value = new Date(year, month, day, hours, minutes);
    } else {
        vmodel.value = null;
    }
}

// --- Computed Component Classes ---

const datepickerWrapperClasses = computed(() => [
    useComputedClass("datepickerWrapperClass", "o-dtpck__date"),
]);

const timepickerWrapperClasses = computed(() => [
    useComputedClass("timepickerWrapperClass", "o-dtpck__time"),
]);
</script>

<template>
    <o-datepicker
        v-if="!isMobileNative || inline"
        ref="datepickerRef"
        v-bind="{ ...$attrs, ...datepicker }"
        v-model="vmodel"
        v-model:active="isActive"
        data-oruga="datetimepicker"
        :class="datepickerWrapperClasses"
        :rounded="rounded"
        :open-on-focus="openOnFocus"
        :position="position"
        :inline="inline"
        :readonly="readonly"
        :expanded="expanded"
        :close-on-click="false"
        :date-formatter="defaultDatetimeFormatter"
        :date-parser="defaultDatetimeParser"
        :min-date="minDate"
        :max-date="maxDate"
        :icon="icon"
        :icon-right="iconRight"
        :icon-right-clickable="iconRightClickable"
        :icon-pack="iconPack"
        :size="datepickerSize"
        :placeholder="placeholder"
        :range="false"
        :disabled="disabled"
        :mobile-native="isMobileNative"
        :locale="locale"
        :teleport="teleport"
        @focus="onFocus"
        @blur="onBlur"
        @change-month="$emit('change-month', $event)"
        @change-year="$emit('change-year', $event)"
        @icon-click="$emit('icon-click', $event)"
        @icon-right-click="$emit('icon-right-click', $event)">
        <template #footer>
            <div :class="timepickerWrapperClasses">
                <o-timepicker
                    ref="timepickerRef"
                    v-bind="timepicker"
                    v-model="vmodel"
                    inline
                    :readonly="readonly"
                    :min-time="minTime"
                    :max-time="maxTime"
                    :size="timepickerSize"
                    :disabled="timepickerDisabled"
                    :mobile-native="isMobileNative"
                    :locale="locale" />
            </div>
            <template v-if="$slots.footer !== undefined">
                <!--
                    @slot Define an additional footer
                -->
                <slot name="footer" />
            </template>
        </template>
    </o-datepicker>

    <!-- Native Picker -->
    <o-input
        v-else
        v-bind="$attrs"
        ref="nativeInputRef"
        type="datetime-local"
        autocomplete="off"
        :value="formatNative(vmodel)"
        :max="formatNative(maxDate)"
        :min="formatNative(minDate)"
        :placeholder="placeholder"
        :size="datepickerSize"
        :icon-pack="iconPack"
        :icon="icon"
        :rounded="rounded"
        :disabled="disabled"
        :readonly="false"
        :use-html5-validation="false"
        @change="onChangeNativePicker"
        @focus="onFocus"
        @blur="onBlur"
        @invalid="onInvalid" />
</template>
