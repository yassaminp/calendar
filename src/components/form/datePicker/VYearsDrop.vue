<template>
    <div class="dropdown mx-3">
        <button
            class="btn btn-secondary dropdown-toggle btn-sm"
            type="button"
            @click="toggleDropdown"
            aria-haspopup="true"
            aria-expanded="false"
        >
            Select Year
        </button>
        {{ isDropdownVisible }}
        <div class="dropdown-menu" v-if="isDropdownVisible" aria-labelledby="dropdownMenuLink" style="display: block">
            <span v-for="(year, i) in availableYears" :key="i" class="dropdown-item" @click="selectYear(year)">
                {{ year }}
            </span>
        </div>
    </div>
</template>

<script>
import { computed, ref } from "vue";

export default {
    props: {
        calendarInfo: {
            type: Object,
            required: true,
        },
    },
    setup(props, { emit }) {
        const isDropdownVisible = ref(false);

        const toggleDropdown = () => {
            isDropdownVisible.value = !isDropdownVisible.value;
        };

        const selectYear = (year) => {
            emit("yearSelected", year);
            isDropdownVisible.value = false;
        };

        const availableYears = computed(() => {
            const years = [];
            const currentYear = props.calendarInfo.year;
            const range = 50;
            for (let i = currentYear - range; i <= currentYear + range; i++) {
                years.push(i);
            }
            return years;
        });

        return {
            isDropdownVisible,
            toggleDropdown,
            selectYear,
            availableYears,
        };
    },
};
</script>

<style>
.dropdown-menu[style] {
    display: block !important;
}
</style>
