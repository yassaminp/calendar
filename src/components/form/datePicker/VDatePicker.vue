<template>
    <div class="d-flex align-items-center justify-content-center p-3 rounded bg-light m-1">
        <div class="align-self-start">
            <select name="calenderType" v-model="selectedCalendarType" @change="generateWeekNames">
                <option value="ghamari">قمری</option>
                <option value="shamsi">شمسی</option>
                <option value="miladi">میلادی</option>
            </select>
            <ul>
                <li v-for="(weekName, index) in weekNames" :key="index">{{ weekName }}</li>
            </ul>
        </div>
    </div>
</template>

<script>
import { ref, onMounted } from "vue";

export default {
    setup() {
        const selectedCalendarType = ref("miladi");
        const weekNames = ref([]);

        const generateWeekNames = () => {};
        onMounted(() => {
            generateWeekNames();
        });

        return {
            selectedCalendarType,
            weekNames,
            generateWeekNames,
        };
    },
};
</script>

<style></style>
