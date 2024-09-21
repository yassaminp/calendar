<template>
    <div class="d-flex align-items-center justify-content-center p-3 rounded bg-light m-1">
        <div class="d-flex flex-column">
            <select name="calenderType" v-model="selectedCalendarType" @change="generateWeekNames" class="align-self-end">
                <option value="ghamari">قمری</option>
                <option value="shamsi">شمسی</option>
                <option value="miladi">میلادی</option>
            </select>

            <ul class="d-flex align-items-center justify-content-center">
                <li v-for="(weekName, index) in weekNames" :key="index" class="px-2">
                    {{ weekName }}
                </li>
            </ul>
        </div>
    </div>
</template>

<script>
import { ref, onMounted } from "vue";
import moment from "moment";
import jMoment from "moment-jalaali";
import "moment-hijri";
export default {
    setup() {
        const selectedCalendarType = ref("miladi");
        const weekNames = ref([]);

        const generateWeekNames = () => {
            switch (selectedCalendarType.value) {
                case "shamsi": {
                    jMoment.loadPersian({ dialect: "persian-modern" });
                    moment.locale("fa");
                    const persianWeekdays = [...jMoment.weekdays()];
                    if (persianWeekdays.length) {
                        const saturday = persianWeekdays.pop();
                        persianWeekdays.unshift(saturday);
                    }
                    weekNames.value = persianWeekdays;
                    break;
                }

                case "ghamari": {
                    moment.locale("ar-sa");
                    weekNames.value = moment.weekdays();
                    break;
                }

                case "miladi": {
                    moment.locale("en");
                    weekNames.value = moment.weekdays();
                    break;
                }

                default: {
                    weekNames.value = moment.weekdays();
                    break;
                }
            }
        };
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
