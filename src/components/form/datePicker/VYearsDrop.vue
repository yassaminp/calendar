<template>
    <div class="dropdown">
        <button class="btn btn-secondary dropdown-toggle" type="button" id="dropdownMenuButton" data-toggle="dropdown">
            <span class="bi-caret-down-fill cursor-pointer px-2"></span>
        </button>
        <div class="dropdown-menu">
            <span v-for="(year, i) in availableYears" :key="i">{{ year }} </span>
        </div>
    </div>
</template>

<script>
import { computed } from "vue";

export default {
    props: {
        calendarInfo: {
            type: Object,
            required: true,
        },
    },
    setup(props) {
        const availableYears = computed(() => {
            const years = [];
            const currentYear = props.calendarInfo.year; // Current year in the selected calendar type
            const range = 50; // You can adjust the range
            for (let i = currentYear - range; i <= currentYear + range; i++) {
                years.push(i);
            }
            return years;
        });

        return {
            availableYears,
        };
    },
};
</script>

<style></style>
