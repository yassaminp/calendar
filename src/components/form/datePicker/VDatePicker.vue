<template>
    <div class="d-flex align-items-center justify-content-center p-3 rounded bg-light m-1">
        <div class="d-flex flex-column container">
            <div class="mb-5 align-self-end">
                <select name="calenderType" v-model="selectedCalendarType" @change="generateCalendar">
                    <option value="ghamari">قمری</option>
                    <option value="shamsi">شمسی</option>
                    <option value="miladi">میلادی</option>
                </select>
            </div>
            <div class="d-flex align-items-center justify-content-between mb-3 mx-2">
                <div>
                    {{ calendarInfo.month }}
                    <input type="" />
                </div>
                <div>
                    <button @click="goPrevMonth">&lt;</button>
                    <button @click="goNextMonth">&gt;</button>
                </div>
            </div>

            <ul class="d-flex align-items-center justify-content-center">
                <li v-for="(weekName, index) in weekNames" :key="index" class="px-2">
                    {{ weekName }}
                </li>
            </ul>
            <div class="container">
                <div v-for="(week, weekIndex) in weekDays" :key="weekIndex" class="row">
                    <div
                        v-for="(day, dayIndex) in week"
                        :key="dayIndex"
                        class="col d-flex align-items-center justify-content-end"
                        :class="dayIndex === 6 ? 'text-danger' : ''"
                    >
                        <span v-if="day">{{ day }}</span>
                        <span v-else class="empty-slot"></span>
                    </div>
                </div>
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
        const selectedCalendarType = ref(null);
        const weekNames = ref([]);
        const timestamp = ref(moment());
        const weekDays = ref([]);

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
        const calculateStartDay = (type) => {
            if (type === "shamsi") {
                const startDay = jMoment(timestamp.value).startOf("jMonth").day();
                return (startDay + 1) % 7;
            } else if (type === "miladi") {
                return moment(timestamp.value).startOf("month").day();
            } else {
                return moment(timestamp.value).startOf("month").day();
            }
        };

        watch(selectedCalendarType, (type) => {
            Object.assign(calendarInfo, {
                startDay: calculateStartDay(type),
                days:
                    type === "shamsi"
                        ? jMoment(timestamp.value).daysInMonth()
                        : type === "miladi"
                        ? moment(timestamp.value).daysInMonth()
                        : moment(timestamp.value).daysInMonth(),
                month:
                    type === "shamsi"
                        ? jMoment(timestamp.value).jMonth()
                        : type === "miladi"
                        ? moment(timestamp.value).month()
                        : moment(timestamp.value).month(),
                year:
                    type === "shamsi"
                        ? jMoment(timestamp.value).jYear()
                        : type === "miladi"
                        ? moment(timestamp.value).year()
                        : moment(timestamp.value).year(),
            });

            fullCalendarDays();
        });

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
        };
    },
};
</script>

<style></style>
