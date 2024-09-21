<template>
    <div class="d-flex align-items-center justify-content-center p-3 rounded bg-light m-1">
        <div class="d-flex flex-column">
            <select name="calenderType" v-model="selectedCalendarType" @change="generateCalendar" class="align-self-end">
                <option value="ghamari">قمری</option>
                <option value="shamsi">شمسی</option>
                <option value="miladi">میلادی</option>
            </select>

            <ul class="d-flex align-items-center justify-content-center">
                <li v-for="(weekName, index) in weekNames" :key="index" class="px-2">
                    {{ weekName }}
                </li>
            </ul>

            <div class="container">
                <div v-for="(week, index) in 6" :key="index" class="row"></div>
            </div>
        </div>
    </div>
</template>

<script>
import { ref, onMounted, reactive, watch } from "vue";
import moment from "moment";
import jMoment from "moment-jalaali";
import "moment-hijri";
export default {
    setup() {
        const selectedCalendarType = ref("miladi");
        const weekNames = ref([]);
        const timestamp = ref(moment());

        const calendarInfo = reactive({
            startDay: "",
            days: "",
            month: "",
            year: "",
        });

        const generateCalendar = () => {
            switch (selectedCalendarType.value) {
                case "shamsi": {
                    jMoment.loadPersian({ dialect: "persian-modern" });
                    moment.locale("fa");
                    const persianWeekdays = [...jMoment.weekdays()];
                    const saturday = persianWeekdays.pop();
                    persianWeekdays.unshift(saturday);
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

        watch(selectedCalendarType, (type) => {
            Object.assign(calendarInfo, {
                startDay:
                    type === "shamsi"
                        ? jMoment(timestamp.value).startOf("jMonth").day()
                        : type === "miladi"
                        ? moment(timestamp.value).startOf("month").day()
                        : moment(timestamp.value).startOf("month").day(),
                month:
                    type === "shamsi"
                        ? jMoment(timestamp.value).jMonth()
                        : type === "miladi"
                        ? moment(timestamp.value).month()
                        : moment(timestamp.value).month(),
            });
            console.log(calendarInfo.startDay);
        });
        onMounted(() => {
            generateCalendar();
        });

        return {
            selectedCalendarType,
            weekNames,
            generateCalendar,
        };
    },
};
</script>

<style></style>
