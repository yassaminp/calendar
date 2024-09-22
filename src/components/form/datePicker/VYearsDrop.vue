<template>
    <div class="dropdown mx-3">
        <button
            class="btn btn-secondary dropdown-toggle btn-sm"
            type="button"
            @click="toggleDropdown"
            aria-haspopup="true"
            aria-expanded="false"
        ></button>

        <div
            class="dropdown-menu scrollable-dropdown"
            ref="dropdownMenu"
            v-if="isDropdownVisible"
            aria-labelledby="dropdownMenuLink"
            style="display: block"
        >
            <span
                v-for="(year, i) in availableYears"
                :key="i"
                class="dropdown-item"
                @click="selectYear(year)"
                :class="{ ' border border-dark rounded active-year': calendarInfo.year === year }"
            >
                {{ year }}
            </span>
        </div>
    </div>
</template>

<script>
import { computed, ref, watch, nextTick } from "vue";

export default {
    props: {
        calendarInfo: {
            type: Object,
            required: true,
        },
    },
    setup(props, { emit }) {
        const isDropdownVisible = ref(false);
        const dropdownMenu = ref(null);

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
            const range = 100;
            for (let i = currentYear - range; i <= currentYear + range; i++) {
                years.push(i);
            }
            return years;
        });

        watch(
            () => isDropdownVisible.value,
            async (newVal) => {
                if (newVal) {
                    await nextTick(); // Wait for the DOM update
                    if (dropdownMenu.value) {
                        const currentYearEl = dropdownMenu.value.querySelector(".active-year");
                        if (currentYearEl) {
                            currentYearEl.scrollIntoView({
                                behavior: "smooth",
                                block: "center",
                            });
                        }
                    }
                }
            }
        );

        return {
            isDropdownVisible,
            toggleDropdown,
            selectYear,
            availableYears,
            dropdownMenu,
        };
    },
};
</script>

<style scoped>
.scrollable-dropdown {
    max-height: 180px;
    overflow-y: auto;
}
</style>
