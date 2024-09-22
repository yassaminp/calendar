<template>
    <div class="dropdown mx-3">
        <button class="btn btn-secondary dropdown-toggle btn-sm" type="button" @click="toggleDropdown"></button>
        <div class="dropdown-menu" v-if="toggleDropdown">
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
        showDropdown: {
            type: Boolean,
            default: false,
        },
    },
    setup(props, context) {
        const toggleDropdown = () => {
            context.emit("toggleDropdown");
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
            availableYears,
            toggleDropdown,
        };
    },
};
</script>

<style></style>
