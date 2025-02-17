<script setup lang="ts">
import { computed, ref, watch, type PropType } from "vue";

import OButton from "../button/Button.vue";
import OSelect from "../select/Select.vue";
import OPickerWrapper from "./PickerWrapper.vue";
import ODatepickerTable from "./DatepickerTable.vue";
import ODatepickerMonth from "./DatepickerMonth.vue";

import { baseComponentProps } from "@/utils/SharedProps";
import { getOption } from "@/utils/config";
import {
    useComputedClass,
    useVModelBinding,
    useMatchMedia,
    usePropBinding,
} from "@/composables";

import { getMonthNames, getWeekdayNames } from "./utils";
import {
    useDatepickerShare,
    type DatepickerEvent,
    type FocusedDate,
} from "./useDatepickerShare";

import type { ComponentClass } from "@/types";

/**
 * An input with a simple dropdown/modal for selecting a date, uses native datepicker for mobile
 * @displayName Datepicker
 * @style _datepicker.scss
 */
defineOptions({
    isOruga: true,
    name: "ODatepicker",
    configField: "datepicker",
});

const props = defineProps({
    // add global shared props (will not be displayed in the docs)
    ...baseComponentProps,
    /** @model */
    modelValue: {
        type: [Date, Array] as PropType<Date | Date[]>,
        default: undefined,
    },
    /** The active state of the dropdown, use v-model:active to make it two-way binding. */
    active: { type: Boolean, default: false },
    /**
     * Define picker mode
     * @values date, month
     */
    type: {
        type: String,
        default: "date",
        validator: (value: string) => ["month", "date"].indexOf(value) >= 0,
    },
    /** Set custom day names, else use names based on locale */
    dayNames: {
        type: Array as PropType<string[]>,
        default: () => getOption("datepicker.dayNames", undefined),
    },
    /** Set custom month names, else use names based on locale */
    monthNames: {
        type: Array as PropType<string[]>,
        default: () => getOption("datepicker.monthNames", undefined),
    },
    /**
     * Size of the control input
     * @values small, medium, large
     */
    size: {
        type: String,
        default: () => getOption("datepicker.size"),
    },
    /** Set default date to focus on */
    focusedDate: { type: Date, default: undefined },
    /** Events to display on picker */
    events: { type: Array as PropType<DatepickerEvent[]>, default: undefined },
    /** Event indicators for style class definiton */
    indicators: { type: String, default: "dots" },
    /** Min date to select */
    minDate: { type: Date, default: undefined },
    /** Max date to select */
    maxDate: { type: Date, default: undefined },
    /** Enable date range selection */
    range: { type: Boolean, default: false },
    /** Makes input full width when inside a grouped or addon field */
    expanded: { type: Boolean, default: false },
    /** Makes the input rounded */
    rounded: { type: Boolean, default: false },
    /** Display datepicker inline */
    inline: { type: Boolean, default: false },
    /** Input placeholder */
    placeholder: { type: String, default: undefined },
    /** Same as native input readonly */
    readonly: { type: Boolean, default: true },
    /** Same as native, also push new item to v-model instead of replacing */
    multiple: { type: Boolean, default: false },
    /** Same as native disabled */
    disabled: { type: Boolean, default: false },
    /** Open dropdown on focus */
    openOnFocus: {
        type: Boolean,
        default: () => getOption("datepicker.openOnFocus", true),
    },
    /** Close dropdown on click */
    closeOnClick: {
        type: Boolean,
        default: () => getOption("datepicker.closeOnClick", true),
    },
    /** Date format locale */
    locale: {
        type: String,
        default: () => getOption("locale"),
    },
    /** Custom function to format a date into a string */
    dateFormatter: {
        type: Function as PropType<(date: Date | Date[]) => string>,
        default: (
            date: Date | Date[],
            defaultFunction: (date: Date | Date[]) => string,
        ) => getOption("datepicker.dateFormatter", defaultFunction)(date),
    },
    /** Custom function to parse a string into a date */
    dateParser: {
        type: Function as PropType<(date: string) => Date>,
        default: (date: string, defaultFunction: (date: string) => Date) =>
            getOption("datepicker.dateParser", defaultFunction)(date),
    },
    /** Date creator function, default is `new Date()` */
    dateCreator: {
        type: Function as PropType<() => Date>,
        default: () => getOption("datepicker.dateCreator", () => new Date())(),
    },
    /** Define a list of dates which can be selected */
    selectableDates: {
        type: [Array, Function] as PropType<Date[] | ((date: Date) => boolean)>,
        default: () => [],
    },
    /** Define a list of dates which can not be selected */
    unselectableDates: {
        type: [Array, Function] as PropType<Date[] | ((date: Date) => boolean)>,
        default: () => [],
    },
    /** Define a list of weeks which can not be selected */
    unselectableDaysOfWeek: {
        type: Array as PropType<number[]>,
        default: () =>
            getOption("datepicker.unselectableDaysOfWeek", undefined),
    },
    /** Show nearby month days */
    nearbyMonthDays: {
        type: Boolean,
        default: () => getOption("datepicker.nearbyMonthDays", true),
    },
    /** Define if nearby month days can be selected */
    nearbySelectableMonthDays: {
        type: Boolean,
        default: () => getOption("datepicker.nearbySelectableMonthDays", false),
    },
    /** Show weeek numbers */
    showWeekNumber: {
        type: Boolean,
        default: () => getOption("datepicker.showWeekNumber", false),
    },
    /** Define if weeek numbers are clickable */
    weekNumberClickable: {
        type: Boolean,
        default: () => getOption("datepicker.weekNumberClickable", false),
    },
    /** Set the first day of a week */
    firstDayOfWeek: {
        type: Number,
        default: () => getOption("datepicker.firstDayOfWeek", 0),
    },
    /** Rules for the first week : 1 for the 1st January, 4 for the 4th January */
    rulesForFirstWeek: { type: Number, default: () => 4 },
    /** Define the range of years to show */
    yearsRange: {
        type: Array as PropType<number[]>,
        default: () => getOption("datepicker.yearsRange", [-100, 10]),
    },
    /** Trap dropdown on focus */
    trapFocus: {
        type: Boolean,
        default: () => getOption("datepicker.trapFocus", true),
    },
    /** Position of the dropdown relative to the input */
    position: { type: String, default: undefined },
    /** Enable dropdown mobile modal */
    mobileModal: {
        type: Boolean,
        default: () => getOption("datepicker.mobileModal", true),
    },
    /** Enable mobile native input if mobile agent */
    mobileNative: {
        type: Boolean,
        default: () => getOption("datepicker.mobileNative", true),
    },
    /**
     * Icon pack to use
     * @values mdi, fa, fas and any other custom icon pack
     */
    iconPack: {
        type: String,
        default: () => getOption("datepicker.iconPack", undefined),
    },
    /** Icon to be shown */
    icon: {
        type: String,
        default: () => getOption("datepicker.icon", undefined),
    },
    /** Icon to be added on the right side */
    iconRight: {
        type: String,
        default: () => getOption("datepicker.iconRight", undefined),
    },
    /** Make the icon right clickable */
    iconRightClickable: { type: Boolean, default: false },
    /** Icon name for previous icon */
    iconPrev: {
        type: String,
        default: () => getOption("datepicker.iconPrev", "chevron-left"),
    },
    /** Icon name for next icon */
    iconNext: {
        type: String,
        default: () => getOption("datepicker.iconNext", "chevron-right"),
    },
    /** Mobile breakpoint as max-width value */
    mobileBreakpoint: {
        type: String,
        default: () => getOption("datepicker.mobileBreakpoint"),
    },
    /**
     * Append the component to another part of the DOM.
     * Set `true` to append the component to the body.
     * In addition, any CSS selector string or an actual DOM node can be used.
     */
    teleport: {
        type: [Boolean, String, Object],
        default: () => getOption("datepicker.teleport", false),
    },
    /** Enable html 5 native validation */
    useHtml5Validation: {
        type: Boolean,
        default: () => getOption("useHtml5Validation", true),
    },
    /** The message which is shown when a validation error occurs */
    validationMessage: { type: String, default: undefined },
    /** Accessibility next button aria label */
    ariaNextLabel: {
        type: String,
        default: () => getOption("datepicker.ariaNextLabel", "Next Page"),
    },
    /** Accessibility previous button aria label  */
    ariaPreviousLabel: {
        type: String,
        default: () => getOption("datepicker.ariaNextLabel", "Previous Page"),
    },
    // class props (will not be displayed in the docs)
    rootClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    sizeClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    boxClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    headerClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    headerButtonsClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    headerButtonsSizeClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    prevButtonClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    nextButtonClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    listsClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    footerClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableHeadClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableHeadCellClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableBodyClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableRowClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableCellClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableCellInvisibleClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableCellSelectedClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableCellFirstSelectedClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableCellWithinSelectedClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableCellLastSelectedClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableCellFirstHoveredClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableCellWithinHoveredClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableCellLastHoveredClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableCellTodayClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableCellSelectableClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableCellUnselectableClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableCellNearbyClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableCellEventsClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableEventsClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableEventClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableEventVariantClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    tableEventIndicatorsClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    monthClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    monthBodyClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    monthTableClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    monthCellClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    monthCellSelectedClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    monthCellFirstSelectedClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    monthCellWithinSelectedClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    monthCellLastSelectedClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    monthCellWithinHoveredRangeClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    monthCellFirstHoveredClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    monthCellWithinHoveredClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    monthCellLastHoveredClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    monthCellTodayClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    monthCellSelectableClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    monthCellUnselectableClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    monthCellEventsClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    mobileClass: {
        type: [String, Array, Function] as PropType<ComponentClass>,
        default: undefined,
    },
    /**
     * Class configuration for the internal input component
     * @ignore
     */
    inputClasses: {
        type: Object,
        default: () => getOption("datepicker.inputClasses", {}),
    },
    /**
     * Class configuration for the internal dropdown component
     * @ignore
     */
    dropdownClasses: {
        type: Object,
        default: () => getOption("datepicker.dropdownClasses", {}),
    },
    /**
     * Class configuration for the internal select component
     * @ignore
     */
    selectClasses: {
        type: Object,
        default: () => getOption("datepicker.selectClasses", {}),
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

const { defaultDateFormatter, defaultDateParser } = useDatepickerShare(props);

const { isMobile } = useMatchMedia(props.mobileBreakpoint);

const vmodel = useVModelBinding<Date | Date[]>(props, emits, { passive: true });

/** Dropdown active state */
const isActive = usePropBinding<boolean>("active", props, emits);

/** modelValue formated into string */
const formattedValue = computed(() =>
    Array.isArray(vmodel.value)
        ? (props.dateFormatter as any)([...vmodel.value], defaultDateFormatter)
        : (props.dateFormatter as any)(vmodel.value, defaultDateFormatter),
);

const isTypeMonth = computed(() => props.type === "month");

/**
 * When v-model is changed:
 *   1. Update internal value.
 *   2. If it's invalid, validate again.
 */
watch(
    () => props.modelValue,
    (value) => {
        // updateInternalState
        if (vmodel.value !== value) {
            const isArray = Array.isArray(value);
            const currentDate = isArray
                ? !value.length
                    ? props.dateCreator()
                    : value[value.length - 1]
                : !value
                  ? props.dateCreator()
                  : value;
            if (
                !isArray ||
                (isArray &&
                    Array.isArray(vmodel.value) &&
                    value.length > vmodel.value.length)
            ) {
                focusedDateData.value = {
                    day: currentDate.getDate(),
                    month: currentDate.getMonth(),
                    year: currentDate.getFullYear(),
                };
            }
        }
    },
);

watch(
    () => props.focusedDate,
    (value) => {
        if (value) {
            focusedDateData.value = {
                day: value.getDate(),
                month: value.getMonth(),
                year: value.getFullYear(),
            };
        }
    },
);

const _initialDate =
    (Array.isArray(props.modelValue)
        ? props.modelValue[0]
        : props.modelValue) ||
    props.focusedDate ||
    props.dateCreator();

if (
    !props.modelValue &&
    props.maxDate &&
    props.maxDate.getFullYear() < _initialDate.getFullYear()
) {
    _initialDate.setFullYear(props.maxDate.getFullYear());
}

const focusedDateData = ref<FocusedDate>({
    day: _initialDate.getDate(),
    month: _initialDate.getMonth(),
    year: _initialDate.getFullYear(),
});

/*
 * Emit input event on month and/or year change
 */
watch(
    () => focusedDateData.value.month,
    (value) => emits("change-month", value),
);
watch(
    () => focusedDateData.value.year,
    (value) => emits("change-year", value),
);

const computedMonthNames = computed(() =>
    Array.isArray(props.monthNames)
        ? props.monthNames
        : getMonthNames(props.locale),
);

const listOfMonths = computed(() => {
    let minMonth = 0;
    let maxMonth = 12;
    if (
        props.minDate &&
        focusedDateData.value.year === props.minDate.getFullYear()
    ) {
        minMonth = props.minDate.getMonth();
    }
    if (
        props.maxDate &&
        focusedDateData.value.year === props.maxDate.getFullYear()
    ) {
        maxMonth = props.maxDate.getMonth();
    }
    return computedMonthNames.value.map((name, index) => ({
        name: name,
        index: index,
        disabled: index < minMonth || index > maxMonth,
    }));
});

const computedDayNames = computed(() => {
    if (Array.isArray(props.dayNames)) return props.dayNames;
    return getWeekdayNames(props.locale);
});

/*
 * Returns an array of years for the year dropdown. If earliest/latest
 * dates are set by props, range of years will fall within those dates.
 */
const listOfYears = computed(() => {
    let latestYear = focusedDateData.value.year + props.yearsRange[1];
    if (props.maxDate && props.maxDate.getFullYear() < latestYear) {
        latestYear = Math.max(
            props.maxDate.getFullYear(),
            focusedDateData.value.year,
        );
    }

    let earliestYear = focusedDateData.value.year + props.yearsRange[0];
    if (props.minDate && props.minDate.getFullYear() > earliestYear) {
        earliestYear = Math.min(
            props.minDate.getFullYear(),
            focusedDateData.value.year,
        );
    }

    return Array.from(
        { length: latestYear - earliestYear || 1 },
        (value, index) => earliestYear + index,
    ).reverse();
});

const showPrev = computed(() => {
    if (!props.minDate) return true;
    if (isTypeMonth.value)
        return focusedDateData.value.year > props.minDate.getFullYear();

    const dateToCheck = new Date(
        focusedDateData.value.year,
        focusedDateData.value.month,
    );
    const date = new Date(
        props.minDate.getFullYear(),
        props.minDate.getMonth(),
    );
    return dateToCheck > date;
});

/**
 * Either decrement month by 1 if not January or decrement year by 1
 * and set month to 11 (December) or decrement year when 'month'
 */
function prev(): void {
    if (props.disabled) return;

    if (isTypeMonth.value) {
        focusedDateData.value.year -= 1;
    } else {
        if (focusedDateData.value.month > 0) {
            focusedDateData.value.month -= 1;
        } else {
            focusedDateData.value.month = 11;
            focusedDateData.value.year -= 1;
        }
    }
}

const showNext = computed(() => {
    if (!props.maxDate) return true;
    if (isTypeMonth.value)
        return focusedDateData.value.year < props.maxDate.getFullYear();

    const dateToCheck = new Date(
        focusedDateData.value.year,
        focusedDateData.value.month,
    );
    const date = new Date(
        props.maxDate.getFullYear(),
        props.maxDate.getMonth(),
    );
    return dateToCheck < date;
});

/**
 * Either increment month by 1 if not December or increment year by 1
 * and set month to 0 (January) or increment year when 'month'
 */
function next(): void {
    if (props.disabled) return;
    if (isTypeMonth.value) {
        focusedDateData.value.year += 1;
    } else {
        if (focusedDateData.value.month < 11) {
            focusedDateData.value.month += 1;
        } else {
            focusedDateData.value.month = 0;
            focusedDateData.value.year += 1;
        }
    }
}

function formatNative(value: Date | Date[]): string {
    if (Array.isArray(value)) value = value[0];

    if (!value) return "";
    const date = new Date(value);

    if (isTypeMonth.value) {
        // Format date into string 'YYYY-MM'
        const year = date.getFullYear();
        const month = date.getMonth() + 1;
        return year + "-" + ((month < 10 ? "0" : "") + month);
    } else {
        // Format date into string 'YYYY-MM-DD'
        const year = date.getFullYear();
        const month = date.getMonth() + 1;
        const day = date.getDate();
        return (
            year +
            "-" +
            ((month < 10 ? "0" : "") + month) +
            "-" +
            ((day < 10 ? "0" : "") + day)
        );
    }
}

// --- Event Handler ---

/** Parse string into date */
function onChange(value: string): void {
    const date = (props.dateParser as any)(value, defaultDateParser);

    if (
        date &&
        Array.isArray(date) &&
        date.length === 2 &&
        !isNaN(date[0]) &&
        !isNaN(date[1])
    ) {
        vmodel.value = date;
    } else {
        vmodel.value = null;
    }
}

/** Parse date from string */
function onChangeNativePicker(value: string): void {
    const s = value ? value.split("-") : [];
    if (s.length === 3) {
        const year = parseInt(s[0], 10);
        const month = parseInt(s[1]) - 1;
        const day = parseInt(s[2]);
        vmodel.value = new Date(year, month, day);
    } else {
        vmodel.value = null;
    }
}

// --- Computed Component Classes ---

const dropdownClass = computed(() =>
    useComputedClass("dropdownClasses.rootClass", "o-dpck__dropdown"),
);

const rootClasses = computed(() => [
    useComputedClass("rootClass", "o-dpck"),
    {
        [useComputedClass("sizeClass", "o-dpck--", props.size)]: props.size,
    },
    {
        [useComputedClass("mobileClass", "o-dpck--mobile")]: isMobile.value,
    },
    {
        [useComputedClass("expandedClass", "o-dpck--expanded")]: props.expanded,
    },
]);

const boxClasses = computed(() => [
    useComputedClass("boxClass", "o-dpck__box"),
]);

const headerClasses = computed(() => [
    useComputedClass("headerClass", "o-dpck__header"),
]);

const headerButtonsClasses = computed(() => [
    useComputedClass("headerButtonsClass", "o-dpck__header__buttons"),
    {
        [useComputedClass(
            "headerButtonsSizeClass",
            "o-dpck__header__buttons--",
            props.size,
        )]: props.size,
    },
]);

const prevButtonClasses = computed(() => [
    useComputedClass("prevButtonClass", "o-dpck__header__previous"),
]);

const nextButtonClasses = computed(() => [
    useComputedClass("nextButtonClass", "o-dpck__header__next"),
]);

const listsClasses = computed(() => [
    useComputedClass("listsClass", "o-dpck__header__list"),
]);

const footerClasses = computed(() => [
    useComputedClass("footerClass", "o-dpck__footer"),
]);
</script>

<template>
    <OPickerWrapper
        ref="wrapperRef"
        v-model:active="isActive"
        data-oruga="datepicker"
        :value="vmodel"
        :picker-props="props"
        :formatted-value="formattedValue"
        :native-type="!isTypeMonth ? 'date' : 'month'"
        :native-value="formatNative(vmodel)"
        :native-max="formatNative(maxDate)"
        :native-min="formatNative(minDate)"
        :stay-open="multiple"
        :dropdown-class="dropdownClass"
        :root-classes="rootClasses"
        :box-class="boxClasses"
        @change="onChange"
        @native-change="onChangeNativePicker"
        @focus="$emit('focus', $event)"
        @blur="$emit('blur', $event)"
        @invalid="$emit('invalid', $event)"
        @icon-click="$emit('icon-click', $event)"
        @icon-right-click="$emit('icon-right-click', $event)">
        <template v-if="$slots.trigger" #trigger>
            <!--
                @slot Override the trigger
            -->
            <slot name="trigger" />
        </template>
        <header :class="headerClasses">
            <!--
                @slot Override the header
            -->
            <slot name="header">
                <div :class="headerButtonsClasses">
                    <OButton
                        v-if="!disabled"
                        :class="prevButtonClasses"
                        :disabled="!showPrev"
                        :aria-label="ariaPreviousLabel"
                        :icon-pack="iconPack"
                        :icon-left="iconPrev"
                        outlined
                        @click.prevent="prev"
                        @keydown.enter.prevent="prev"
                        @keydown.space.prevent="prev" />

                    <OButton
                        v-if="!disabled"
                        :class="nextButtonClasses"
                        :disabled="!showNext"
                        :aria-label="ariaNextLabel"
                        :icon-pack="iconPack"
                        :icon-left="iconNext"
                        outlined
                        @click.prevent="next"
                        @keydown.enter.prevent="next"
                        @keydown.space.prevent="next" />

                    <div :class="listsClasses">
                        <o-select
                            v-if="!isTypeMonth"
                            v-model="focusedDateData.month"
                            :disabled="disabled"
                            :size="size"
                            v-bind="selectClasses">
                            <option
                                v-for="month in listOfMonths"
                                :key="month.name"
                                :value="month.index"
                                :disabled="month.disabled">
                                {{ month.name }}
                            </option>
                        </o-select>
                        <o-select
                            v-model="focusedDateData.year"
                            :disabled="disabled"
                            :size="size"
                            v-bind="selectClasses">
                            <option
                                v-for="year in listOfYears"
                                :key="year"
                                :value="year">
                                {{ year }}
                            </option>
                        </o-select>
                    </div>
                </div>
            </slot>
        </header>
        <!--
            @slot Override the body
        -->
        <slot name="body">
            <o-datepicker-month
                v-if="isTypeMonth"
                v-model="vmodel"
                v-model:focused-date="focusedDateData"
                :month-names="computedMonthNames"
                :picker-props="props"
                @range-start="(date) => $emit('range-start', date)"
                @range-end="(date) => $emit('range-end', date)" />
            <o-datepicker-table
                v-else
                v-model="vmodel"
                v-model:focused-date="focusedDateData"
                :day-names="computedDayNames"
                :month-names="computedMonthNames"
                :picker-props="props"
                @range-start="(date) => $emit('range-start', date)"
                @range-end="(date) => $emit('range-end', date)" />
        </slot>
        <footer v-if="$slots.footer" :class="footerClasses">
            <!--
                @slot Define an additional footer
            -->
            <slot name="footer" />
        </footer>
    </OPickerWrapper>
</template>
