<template>
    <div class="d-flex align-items-center flex-column justify-content-center p-3 rounded bg-light m-1 p-2 calendar-container">
        <div class="d-flex flex-column">
            <div class="mb-3 d-flex align-items-center justify-content-between">
                <button type="button" class="btn btn-outline-secondary btn-sm" @click="goToToday">برو به امروز</button>
                <select
                    name="calenderType"
                    class="form-select form-select-sm"
                    style="max-width: 100px"
                    v-model="selectedCalendarType"
                    @change="generateCalendar"
                >
                    <option value="ghamari">قمری</option>
                    <option value="shamsi">شمسی</option>
                    <option value="miladi">میلادی</option>
                </select>
            </div>
            <div class="d-flex align-items-center justify-content-between mb-3 mx-2">
                <button @click="goPrevMonth">&lt;</button>
                <div class="d-flex align-items-center justify-content-between">
                    {{
                        selectedCalendarType === "shamsi"
                            ? moment(timestamp).format("jMMMM")
                            : selectedCalendarType === "miladi"
                            ? moment(timestamp).format("MMMM")
                            : hMoment(timestamp).format("iMMMM")
                    }}
                    - {{ calendarInfo.year }}
                    <VYearsDrop :calendarInfo="calendarInfo" @yearSelected="handleYearSelection" />
                </div>
                <button @click="goNextMonth">&gt;</button>
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
                            <VDate
                                :day="day"
                                :weekend="dayIndex === 6 ? true : false"
                                :holiday="isHoliday(day, calendarInfo.month, calendarInfo.year) ? true : false"
                                :specialDay="isSpecial(day, calendarInfo.month) ? 'bi-award' : ''"
                            />
                        </div>
                        <span v-else class="empty-slot"></span>
                    </div>
                </div>
            </div>
        </div>
        {{ pickedDate.day ? moment(`${pickedDate.year}-${pickedDate.month}-${pickedDate.day}`, "YYYY-MM-DD").format("YYYY/MM/DD") : "" }}
    </div>
</template>

<script>
import { ref, onMounted, reactive, watch } from "vue";
import moment from "moment-jalaali";
import hMoment from "moment-hijri";
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
        const timestamp = ref();
        const weekDays = ref([]);
        const pickedDate = reactive({
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

        const holidays = ref([
            { year: 1403, month: 1, day: 1, type: "shamsi" },
            { year: 1403, month: 7, day: 3, type: "shamsi" },
            { year: 1403, month: 8, day: 14, type: "shamsi" },
            { year: 1403, month: 10, day: 22, type: "shamsi" },
            { year: 2024, month: 8, day: 25, type: "miladi" },
            { year: 2024, month: 9, day: 20, type: "miladi" },
            { year: 2024, month: 11, day: 2, type: "miladi" },
        ]);
        const birthday = ref({ month: 9, day: 29, type: "miladi" });

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
                    const hijriWeekdays = ["الأحد", "الاثنين", "الثلاثاء", "الأربعاء", "الخميس", "الجمعة", "السبت"];
                    weekNames.value = hijriWeekdays;

                    today.date = hMoment(timestamp).iDate();
                    today.month = hMoment(timestamp).iMonth();
                    today.year = hMoment(timestamp).iYear();
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
                return hMoment(timestamp.value).startOf("iMonth").day();
            }
        };

        watch(selectedCalendarType, (type) => {
            updateCalendar(type);
        });

        watch(timestamp, () => {
            updateCalendar(selectedCalendarType.value);
        });

        const updateCalendar = (type) => {
            pickedDate.day = null;
            pickedDate.month = null;
            pickedDate.year = null;

            Object.assign(calendarInfo, {
                startDay: calculateStartDay(type),
                days:
                    type === "shamsi"
                        ? moment(timestamp.value).daysInMonth()
                        : type === "miladi"
                        ? moment(timestamp.value).daysInMonth()
                        : hMoment(timestamp.value).daysInMonth(),
                month:
                    type === "shamsi"
                        ? moment(timestamp.value).jMonth()
                        : type === "miladi"
                        ? moment(timestamp.value).month()
                        : hMoment(timestamp.value).iMonth(),
                year:
                    type === "shamsi"
                        ? moment(timestamp.value).jYear()
                        : type === "miladi"
                        ? moment(timestamp.value).year()
                        : hMoment(timestamp.value).iYear(),
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
            } else if (selectedCalendarType.value === "miladi") {
                timestamp.value = moment(timestamp.value).subtract(1, "month");
            } else {
                timestamp.value = hMoment(timestamp.value).subtract(1, "iMonth");
            }
            updateCalendar(selectedCalendarType.value);
        };

        const goNextMonth = () => {
            if (selectedCalendarType.value === "shamsi") {
                timestamp.value = moment(timestamp.value).add(1, "jMonth");
            } else if (selectedCalendarType.value === "miladi") {
                timestamp.value = moment(timestamp.value).add(1, "month");
            } else {
                timestamp.value = hMoment(timestamp.value).add(1, "iMonth");
            }
            updateCalendar(selectedCalendarType.value);
        };

        const goToToday = () => {
            timestamp.value = moment();
        };

        const handleYearSelection = (year) => {
            calendarInfo.year = year;
            timestamp.value =
                selectedCalendarType.value === "shamsi"
                    ? moment(timestamp.value).jYear(year).format()
                    : selectedCalendarType.value === "miladi"
                    ? moment(timestamp.value).year(year).format()
                    : hMoment(timestamp.value).iYear(year).format();
        };

        const selectDate = (day) => {
            pickedDate.type = selectedCalendarType.value;
            pickedDate.day = day;
            pickedDate.month = calendarInfo.month;
            pickedDate.year = calendarInfo.year;
        };

        const isHoliday = (day, month, year) => {
            if (!day) return false;

            return holidays.value.some(
                (holiday) =>
                    holiday.day === day && holiday.month === month && holiday.year === year && holiday.type === selectedCalendarType.value
            );
        };

        const isSpecial = (day, month) => {
            if (!day) return false;

            return birthday.value.day === day && birthday.value.month === month && birthday.value.type === selectedCalendarType.value;
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
            goToToday,
            isHoliday,
            isSpecial,
            hMoment,
        };
    },
};
</script>

<style scoped lang="scss">
.calendar-container {
    height: min-content;
}
</style>
