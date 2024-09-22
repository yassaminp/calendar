<template>
    <div class="d-flex align-items-center justify-content-center p-3 rounded bg-light m-1 p-2 calendar-container">
        <div class="d-flex flex-column container">
            <div class="mb-3 align-self-end">
                <select name="calenderType" class="form-select form-select-sm" v-model="selectedCalendarType" @change="generateCalendar">
                    <option value="ghamari">قمری</option>
                    <option value="shamsi">شمسی</option>
                    <option value="miladi">میلادی</option>
                </select>
            </div>
            <div class="d-flex align-items-center justify-content-between mb-3 mx-2">
                <div class="d-flex align-items-center justify-content-between">
                    {{ selectedCalendarType === "shamsi" ? moment(timestamp).format("jMMMM") : moment(timestamp).format("MMMM") }}
                    - {{ calendarInfo.year }}
                    <VYearsDrop :calendarInfo="calendarInfo" @yearSelected="handleYearSelection" />
                </div>
                <div class="buttons-wrapper d-flex align-items-center justify-content-between">
                    <button @click="goPrevMonth">&lt;</button>

                    <button @click="goNextMonth">&gt;</button>
                </div>
            </div>

            <ul class="d-flex align-items-center justify-content-between">
                <li v-for="(weekName, index) in weekNames" :key="index" class="px-1">
                    {{ weekName }}
                </li>
            </ul>
            <div class="container">
                <div v-for="(week, weekIndex) in weekDays" :key="weekIndex" class="row">
                    <div
                        v-for="(day, dayIndex) in week"
                        :key="dayIndex"
                        class="col d-flex align-items-center justify-content-center rounded"
                        :class="{
                            'border border-dark ':
                                today.date === day && today.month === calendarInfo.month && today.year === calendarInfo.year,
                            'text-light bg-dark':
                                pickedDate.day === day && pickedDate.month === calendarInfo.month && pickedDate.year === calendarInfo.year,
                        }"
                    >
                        <div v-if="day" class="cursor-pointer" @click="selectDate(day)">
                            <VDate :day="day" :holiday="dayIndex === 6 ? true : false" />
                        </div>
                        <span v-else class="empty-slot"></span>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import { ref, onMounted, reactive, watch } from "vue";
import moment from "moment-jalaali";
import "moment-hijri";
import VDate from "./VDate.vue";
import VYearsDrop from "./VYearsDrop.vue";
export default {
    components: {
        VDate,
        VYearsDrop,
    },
    setup() {
        const selectedCalendarType = ref(null);
        const weekNames = ref([]);
        const timestamp = ref(moment());
        const weekDays = ref([]);
        const pickedDate = reactive({
            type: null,
            day: null,
            month: null,
            year: null,
        });

        const calendarInfo = reactive({
            startDay: "",
            days: "",
            month: "",
            year: "",
        });

        const today = reactive({
            date: "",
            month: "",
            year: "",
        });

        const generateCalendar = () => {
            switch (selectedCalendarType.value) {
                case "shamsi": {
                    moment.locale("fa");
                    moment.loadPersian({ dialect: "persian-modern" });
                    const persianWeekdays = [...moment.weekdays()];
                    const saturday = persianWeekdays.pop();
                    persianWeekdays.unshift(saturday);
                    weekNames.value = persianWeekdays;

                    today.date = moment(timestamp).jDate();
                    today.month = moment(timestamp).jMonth();
                    today.year = moment(timestamp).jYear();
                    break;
                }

                case "ghamari": {
                    moment.locale("ar-sa");
                    weekNames.value = moment.weekdays();

                    today.date = moment(timestamp).date();
                    today.month = moment(timestamp).month();
                    today.year = moment(timestamp).year();
                    break;
                }

                case "miladi": {
                    moment.locale("en");
                    weekNames.value = moment.weekdays();

                    today.date = moment(timestamp).date();
                    today.month = moment(timestamp).month();
                    today.year = moment(timestamp).year();

                    break;
                }

                default: {
                    weekNames.value = moment.weekdays();
                    today.date = moment(timestamp).date();
                    today.month = moment(timestamp).month();
                    today.year = moment(timestamp).year();
                    break;
                }
            }
        };
        const calculateStartDay = (type) => {
            if (type === "shamsi") {
                const startDay = moment(timestamp.value).startOf("jMonth").day();
                return (startDay + 1) % 7;
            } else if (type === "miladi") {
                return moment(timestamp.value).startOf("month").day();
            } else {
                return moment(timestamp.value).startOf("month").day();
            }
        };

        watch(selectedCalendarType, (type) => {
            updateCalendar(type);
        });

        watch(timestamp, () => {
            updateCalendar(selectedCalendarType.value);
        });

        const updateCalendar = (type) => {
            Object.assign(calendarInfo, {
                startDay: calculateStartDay(type),
                days:
                    type === "shamsi"
                        ? moment(timestamp.value).daysInMonth()
                        : type === "miladi"
                        ? moment(timestamp.value).daysInMonth()
                        : moment(timestamp.value).daysInMonth(),
                month:
                    type === "shamsi"
                        ? moment(timestamp.value).jMonth()
                        : type === "miladi"
                        ? moment(timestamp.value).month()
                        : moment(timestamp.value).month(),
                year:
                    type === "shamsi"
                        ? moment(timestamp.value).jYear()
                        : type === "miladi"
                        ? moment(timestamp.value).year()
                        : moment(timestamp.value).year(),
            });

            fullCalendarDays();
        };

        const fullCalendarDays = () => {
            const totalSlots = 42;
            const daysArray = [];

            for (let i = 0; i < calendarInfo.startDay; i++) {
                daysArray.push(null);
            }

            for (let i = 1; i <= calendarInfo.days; i++) {
                daysArray.push(i);
            }

            while (daysArray.length < totalSlots) {
                daysArray.push(null);
            }
            weekDays.value = [];
            for (let i = 0; i < daysArray.length; i += 7) {
                weekDays.value.push(daysArray.slice(i, i + 7));
            }
            return weekDays.value;
        };

        const goPrevMonth = () => {
            if (selectedCalendarType.value === "shamsi") {
                timestamp.value = moment(timestamp.value).subtract(1, "jMonth");
            } else {
                timestamp.value = moment(timestamp.value).subtract(1, "month");
            }
            updateCalendar(selectedCalendarType.value);
        };

        const goNextMonth = () => {
            if (selectedCalendarType.value === "shamsi") {
                timestamp.value = moment(timestamp.value).add(1, "jMonth");
            } else {
                timestamp.value = moment(timestamp.value).add(1, "month");
            }
            updateCalendar(selectedCalendarType.value);
        };

        const handleYearSelection = (year) => {
            calendarInfo.year = year;
            timestamp.value =
                selectedCalendarType.value === "shamsi"
                    ? moment(timestamp.value).jYear(year).format()
                    : moment(timestamp.value).year(year).format();
        };

        const selectDate = (day) => {
            pickedDate.type = selectedCalendarType.value;
            pickedDate.day = day;
            pickedDate.month = calendarInfo.month;
            pickedDate.year = calendarInfo.year;
        };

        onMounted(() => {
            generateCalendar();
            selectedCalendarType.value = "miladi";
        });

        return {
            selectedCalendarType,
            weekNames,
            generateCalendar,
            weekDays,
            calendarInfo,
            moment,
            timestamp,
            goNextMonth,
            goPrevMonth,
            today,
            selectDate,
            handleYearSelection,
            pickedDate,
        };
    },
};
</script>

<style scoped lang="scss">
.buttons-wrapper {
    width: 20%;
    button {
        background-color: transparent;
        outline: none;
        border: none;
        cursor: pointer;
        font-size: 1.2rem;
        font-weight: 500;
    }
}
.calendar-container {
    height: min-content;
}
</style>
