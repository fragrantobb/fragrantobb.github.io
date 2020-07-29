<template>
    <article>
        <p>{{ currYear }} 년 / {{ currMonth }} 월</p>
        <div>
            <button
                @click="horizontalArrow(true)"
                type="button"
            ><</button>
            <div>
                <table v-show="calendarType === 'yearList'">
                    <tbody>
                        <tr
                            v-for="(bundle, idx) in currYearBundleList"
                            :key="idx"
                        >
                            <td
                                v-for="item in bundle"
                                :key="item"
                            >
                                <button
                                    @click="toMonthList(item)"
                                    type="button">{{ item }}</button>
                            </td>
                        </tr>
                    </tbody>
                </table>
                <table v-show="calendarType === 'monthList'">
                    <tbody>
                        <tr
                            v-for="(bundle, idx) in currMonthBundleList"
                            :key="idx"
                        >
                            <td
                                v-for="item in bundle"
                                :key="item"
                            >
                                <button
                                    @click="toMonthlyDateList(item)"
                                    type="button">{{ item }}</button>
                            </td>
                        </tr>
                    </tbody>
                </table>
                <table v-show="calendarType === 'monthlyDateList'">
                    <thead>
                        <tr>
                            <th
                                v-for="item in currDayList"
                                :key="item"
                            >{{ item }}</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr
                            v-for="(week, idx) in currWeekList"
                            :key="idx"
                        >
                            <td
                                v-for="date in week"
                                :key="date.id"
                            >
                                <p>{{ date.value }}</p>
                                <ul>
                                    <li></li>
                                </ul>
                                <button
                                    v-if="date.value"
                                    type="button"
                                    @click="toggleSubmitModal(true, date.id)"
                                >등록</button>
                            </td>
                        </tr>
                    </tbody>
                </table>
                <table v-show="calendarType === 'weeklyDateList'">
                    <thead>
                        <tr>
                            <th
                                v-for="day in currDayList"
                                :key="day"
                            >{{ day }}</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td
                                v-for="item in weekCalendarItemList"
                                :key="item.id"
                            >
                                <p>{{ item.value }}</p>
                                <button
                                    v-if="item.value"
                                    type="button"
                                    @click="toggleSubmitModal(true, item.id)"
                                >등록</button>
                            </td>
                        </tr>
                    </tbody>
                </table>
                <table v-show="calendarType === 'dateDetail'">
                    <thead>
                        <tr>
                            <th>{{ currDate }} ({{ currDay }})</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td></td>
                        </tr>
                        <tr>
                            <td>
                                <button
                                    type="button"
                                    @click="toggleSubmitModal(true, `${currYear}. ${currMonth}. ${currDate}.`)"
                                >등록</button>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
            <button
                @click="horizontalArrow"
                type="button"
            >></button>
        </div>
        <ul>
            <li>
                <button
                    @click="toYearList"
                    type="button"
                >연 목록</button>
            </li>
            <li>
                <button
                    @click="toMonthList"
                    type="button"
                >연</button>
            </li>
            <li>
                <button
                    @click="toMonthlyDateList"
                    type="button"
                >월</button>
            </li>
            <li>
                <button
                    @click="toWeeklyDateList"
                    type="button"
                >주</button>
            </li>
            <li>
                <button
                    @click="toDateDetail"
                    type="button"
                >오늘</button>
            </li>
        </ul>
        <Modal
            :active="activeModal"
            @toggle="toggleSubmitModal"
        >
            <InsertLedger @toggle="toggleSubmitModal" />
        </Modal>
    </article>
</template>

<script>
import utils from '~/assets/js/utils.js';
import Modal from '~/components/modal/Modal.vue';
import InsertLedger from '~/components/modal/InsertLedger.vue';
import { ref, computed, onMounted, onUnmounted, watch, reactive } from '@vue/composition-api';

export default {
    name: 'Ledger',
    components: { Modal, InsertLedger },
    setup(props, { attrs, emit, isServer, listeners, parent, refs, root, slots, ssrContext }) {
        const toPureDate = dateObj => {
            dateObj.setHours(0);
            dateObj.setMinutes(0);
            dateObj.setSeconds(0);
        };
        const query = root.$route.query;
        const calendarTypeList = ['yearList', 'monthList', 'monthlyDateList', 'weeklyDateList', 'dateDetail'];
        const calendarType = ref(null);
        const currMillis = ref((() => {
            const now = new Date();
            if (query.time) {
                now.setTime(query.time);
            }
            toPureDate(now);
            return now.getTime();
        })());
        const commitData = changes => {
            const query = utils.deepCopy(root.$route.query);
            if (changes.hasOwnProperty('type')) {
                query.type = changes.type;
                calendarType.value = changes.type;
            }
            if (changes.hasOwnProperty('time')) {
                query.time = changes.time;
                currMillis.value = changes.time;
            }
            root.$router.replace({ query });
        };
        (() => {
            if (calendarTypeList.some(type => type === query.type)) {
                calendarType.value = query.type;
            } else {
                calendarType.value = 'monthlyDateList';
                commitData({ type: 'monthlyDateList' });
            }
        })();
        const typeMaxLevel = calendarTypeList.length - 1;
        const typeLevel = computed(() => calendarTypeList.indexOf(calendarType.value));
        const currYear = computed(() => new Date(currMillis.value).getFullYear());
        const currMonth = computed(() => new Date(currMillis.value).getMonth() + 1);
        const currDate = computed(() => new Date(currMillis.value).getDate());
        const currYearBundleList = computed(() => {
            const yearBundleList = [];
            let yearBundle = [];
            let idx = 1;
            for (let cnt = currYear.value - 5; cnt <= currYear.value + 6; cnt++) {
                yearBundle.push(cnt);
                if (idx !== 1 && !(idx % 4)) {
                    yearBundleList.push(yearBundle);
                    yearBundle = [];
                }
                idx++;
            }
            return yearBundleList;
        });
        const currMonthBundleList = [[1, 2, 3, 4], [5, 6, 7, 8], [9, 10 ,11, 12]];
        const currDayList = ['월', '화', '수', '목', '금', '토', '일'];
        const currWeekList = computed(() => {
            const getDay = originDay => originDay === 0 ? 6 : originDay - 1;
            const weekList = [];
            let week = [];
            const currDateObj = new Date(currMillis.value);
            const firstDate = new Date(currDateObj.getFullYear(), currDateObj.getMonth(), 1);
            const lastDate = new Date(currDateObj.getFullYear(), currDateObj.getMonth() + 1, 0);
            for (let date = firstDate.getDate(); date <= lastDate.getDate(); date++) {
                firstDate.setDate(date);
                const day = getDay(firstDate.getDay());
                if (date === 1 && day !== 0) {
                    const tempDateObj = new Date(currMillis.value);
                    tempDateObj.setDate(0);
                    for (let cnt = day - 1; cnt >= 0; cnt--) {
                        week.unshift({ id: tempDateObj.toLocaleDateString(), value: tempDateObj.getDate() });
                        tempDateObj.setDate(tempDateObj.getDate() - 1);
                    }
                }
                week.push({ id: firstDate.toLocaleDateString(), value: firstDate.getDate() });
                if (day !== 6 && date === lastDate.getDate()) {
                    const tempDateObj = new Date(currMillis.value);
                    tempDateObj.setMonth(tempDateObj.getMonth());
                    tempDateObj.setDate(1);
                    for (let cnt = day; cnt < 6; cnt++) {
                        week.push({ id: tempDateObj.toLocaleDateString(), value: tempDateObj.getDate() });
                        tempDateObj.setDate(tempDateObj.getDate() + 1);
                    }
                    weekList.push(week);
                    week = [];
                } else if (day === 6 || date === lastDate.getDate()) {
                    weekList.push(week);
                    week = [];
                }
            }
            return weekList;
        });
        const weekCalendarItemList = computed(() => {
            const millis = new Date(currMillis.value).toLocaleDateString();
            const currWeek = currWeekList.value.filter(week => week.some(({ id }) => id === millis))[0];
            return currWeek;
        });
        const currDay = computed(() => {
            const day = new Date(currMillis.value).getDay();
            const parsedDay = day === 0 ? 6 : day - 1;
            return currDayList[parsedDay];
        });
        const keyEvent = () => {
            const horizontalArrow = isLeft => {
                const targetDateObj = new Date(currMillis.value);
                if (calendarType.value === 'yearList') {
                    targetDateObj.setFullYear(targetDateObj.getFullYear() + (isLeft ? -12 : 12));
                } else if (calendarType.value === 'monthList') {
                    targetDateObj.setFullYear(targetDateObj.getFullYear() + (isLeft ? -1 : 1));
                } else if (calendarType.value === 'monthlyDateList') {
                    if (isLeft ? (currMonth.value < 2) : (currMonth.value === 12)) {
                        targetDateObj.setFullYear(targetDateObj.getFullYear() + (isLeft ? -1 : 1));
                        targetDateObj.setMonth(isLeft ? 11 : 0);
                    } else {
                        targetDateObj.setMonth(targetDateObj.getMonth() + (isLeft ? -1 : 1));
                    }
                    const currDateObj = new Date(currMillis.value);
                    if (targetDateObj.getDate() !== currDateObj.getDate()) {
                        targetDateObj.setDate(1);
                        targetDateObj.setMonth(currDateObj.getMonth() + (isLeft ? 0 : 2));
                        targetDateObj.setDate(0);
                        
                    }
                } else if (calendarType.value === 'weeklyDateList') {
                    targetDateObj.setDate(targetDateObj.getDate() + (isLeft ? -7 : 7));
                } else if (calendarType.value === 'dateDetail') {
                    targetDateObj.setDate(targetDateObj.getDate() + (isLeft ? -1 : 1));
                }
                commitData({ time: targetDateObj.getTime() });
            };
            const cb = e => {
                const key = e.key;
                switch (key) {
                    case 'ArrowLeft':
                    case 'ArrowRight':
                        e.preventDefault();
                        horizontalArrow(key === 'ArrowLeft');
                        break;
                    case 'ArrowUp':
                        if (typeLevel.value === 0) return;
                        e.preventDefault();
                        commitData({ type: calendarTypeList[typeLevel.value - 1] });
                        break;
                    case 'ArrowDown':
                        if (typeLevel.value === typeMaxLevel) return;
                        e.preventDefault();
                        commitData({ type: calendarTypeList[typeLevel.value + 1] });
                        break;
                }
            };
            const setArrowKeyEvent = () => {
                if (!document) return false;
                document.addEventListener('keydown', cb);
            };
            const removeKeyEvent = () => {
                if (!document) return false;
                document.removeEventListener('keydown', cb);
            };
            return { setArrowKeyEvent, removeKeyEvent, horizontalArrow };
        };
        const { setArrowKeyEvent, removeKeyEvent, horizontalArrow } = keyEvent();
        const activeModal = ref(false);
        const toggleSubmitModal = (bool, dateStr) => {
            const millisStr = !bool || `${new Date(dateStr).getTime()}`;
            activeModal.value = bool;
        };
        const toYearList = () => {
            commitData({ type: 'yearList' });
        };
        const toMonthList = targetYear => {
            const commitObj = { type: 'monthList' };
            if (typeof targetYear === 'number') {
                commitObj.time = new Date(currMillis.value).setFullYear(targetYear);
            }
            commitData(commitObj);
        };
        const toMonthlyDateList = targetMonth => {
            const commitObj = { type: 'monthlyDateList' };
            if (typeof targetMonth === 'number') {
                commitObj.time = new Date(currMillis.value).setMonth(targetMonth - 1);
            }
            commitData(commitObj);
        };
        const toWeeklyDateList = () => {
            commitData({ type: 'weeklyDateList' });
        };
        const toDateDetail = () => {
            const newDate = new Date();
            toPureDate(newDate);
            commitData({ time: newDate.getTime(), type: 'dateDetail' });
        };

        onMounted(() => {
            setArrowKeyEvent();
        });

        onUnmounted(() => {
            removeKeyEvent();
        });

        return {
            calendarType,
            currMillis,
            currYear,
            currMonth,
            currDayList,
            currWeekList,
            horizontalArrow,
            activeModal,
            toggleSubmitModal,
            toDateDetail,
            currMonthBundleList,
            currYearBundleList,
            weekCalendarItemList,
            currDate,
            currDay,
            toYearList,
            toMonthList,
            toMonthlyDateList,
            toWeeklyDateList,
        };
    },
};
</script>

<style>

</style>
