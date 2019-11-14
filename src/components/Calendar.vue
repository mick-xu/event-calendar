<template>
<div class="calendar">
    <div class="calendar-header">
        <div class="month">{{year}}年{{month+1}}月</div>
        <div class="operation">
            <a class="back" v-show="visible" @click="change(0)">今日</a>
            <i class="btn arrow-left" @click="change(-1)"></i>
            <i class="btn arrow-right" @click="change(1)"></i>
        </div>
    </div>
    <div class="weeks">
        <div class="week" v-for="week in weeks" :key="week">{{week}}</div>
    </div>
    <div class="days" @click="setDay">
        <div class="a-week" v-for='row in days' :key="row.index">
            <div class="day" v-for='day in row' :key="day.day" :data-p="[day.row,day.column]" :class='day.className' :style="{height:`${height}px`,lineHeight:`${height}px`}">{{day.day}}</div>
        </div>
    </div>
</div>
</template>

<script>
export default {
    name: 'Calendar',
    props: {
        value: {
            type: Object,
            default: () => {}
        },
        evnets: {
            type: Array,
            default: () => [
                12, 13, 14, 22, 23
            ]
        }
    },
    data() {
        return {
            year: Number,
            month: Number,
            today: 1,
            weeks: ["一", "二", "三", "四", "五", "六", "日"],
            days: [],
            currentWeek: [],
            visible: false,
            dayPos: [1, 1],
            height: 30
        }
    },
    methods: {
        initCalendar() {
            let today = new Date();
            this.year = today.getFullYear();
            this.month = today.getMonth();
            let day = today.getDate();
            this.initDays();
        },
        initDays() {
            let today = new Date();
            let year = today.getFullYear(),
                month = today.getMonth(),
                day = today.getDate();
            let firstDay = new Date(this.year, this.month, 1),
                dayOfWeek = firstDay.getDay(),
                daysOfFeb = 28 + (this.isLeap(this.year) ? 1 : 0),
                daysOfMonth = [31, daysOfFeb, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
            if (dayOfWeek === 0) dayOfWeek = 7;
            let rows = Math.ceil((dayOfWeek + daysOfMonth[this.month]) / 7);
            this.days = [];
            for (let index = 0; index < rows; index++) {
                let row = new Array();
                for (let week = 1; week < 8; week++) {
                    let idx = 7 * index + week;
                    let date = idx - dayOfWeek + 1;
                    let dayClass = {
                        disabled: false,
                        selected: false,
                        active: false,
                        event: false,
                    };
                    let dayData = 0;
                    let curMonth = this.month + 1;
                    let curYear = this.year;
                    if (date < 1) {
                        let prevMonth = this.month === 0 ? 11 : this.month - 1;
                        let prevYear = this.month === 0 ? this.year - 1 : this.year;
                        dayData = date + daysOfMonth[prevMonth];
                        dayClass.disabled = true;
                        curMonth = prevMonth + 1;
                        curYear = prevYear;
                    } else if (date > daysOfMonth[this.month]) {
                        let nextMonth = this.month === 11 ? 0 : this.month + 1;
                        let nextYear = this.month === 11 ? this.year + 1 : this.year;
                        dayData = date - daysOfMonth[this.month];
                        dayClass.disabled = true;
                        curMonth = nextMonth + 1;
                        curYear = nextYear;
                    } else {
                        dayData = date;
                        if (year === this.year && month === this.month && dayData === day) {
                            dayClass.active = true;
                            this.today = dayData;
                        }
                    }
                    row.push({
                        row: index,
                        column: week,
                        day: dayData,
                        month: curMonth,
                        year: curYear,
                        monthEnd: daysOfMonth[this.month],
                        className: dayClass
                    });
                }
                this.days.push(row);
            }
            this.getValue();
        },
        getValue() {
            let action;
            let week = this.days.filter(week => week.some(day => day.className.selected))[0];
            if (week && week.length > 0) {
                action = "selectedDay";
            } else {
                week = this.days.filter(week => week.some(day => day.className.active))[0];
                if (week) {
                    action = "init";
                } else {
                    week = this.days[0];
                    action = "changeMonth";
                }
            }
            let day, days = week.filter(item => item.className.selected || item.className.active);
            if (days && days.length === 1) {
                day = days[0];
                action = day.className.selected ? "selectedDay" : "init";
            } else if (days && days.length === 2) {
                day = days.filter(item => item.className.selected)[0];
                action = "selectedDay";
            } else {
                day = week.filter(item => !item.className.disabled)[0];
                action = "changeMonth";
            }
            if (week) {
                this.currentWeek = week.map(item => {
                    return {
                        day: `${item.year}-${item.month}-${item.day}`,
                        event: item.className.event
                    }
                });
                let value = {
                    today: `${day.year}-${day.month}-${day.day}`,
                    event: day.className.event,
                    week: this.currentWeek,
                    action: action
                }
                this.$emit("input", value);
            }
        },
        isLeap(year) {
            return year % 4 == 0 ?
                year % 100 != 0 ?
                true :
                year % 400 == 0 ?
                true :
                false :
                false;
        },
        setDay(event) {
            let target = event.target;
            if (target.className === "disabled") {
                return;
            }
            this.days[this.dayPos[0]][this.dayPos[1] - 1].className.selected = false;
            let attr = target.getAttribute("data-p");
            this.dayPos = JSON.parse(`[${attr}]`);
            let selectedDay = this.days[this.dayPos[0]][this.dayPos[1] - 1];
            selectedDay.className.selected = true;
            let day = target.innerText;
            this.today = parseInt(day);
            this.getValue(this.days[this.dayPos[0]], selectedDay);
        },
        change(step) {
            if (step === 0) {
                let today = new Date();
                this.year = today.getFullYear();
                this.month = today.getMonth();
                this.visible = false;
                this.initDays();
            } else {
                this.visible = true;
                this.month += step;
                if (this.month > 11) {
                    this.month = 0;
                    this.year++;
                } else if (this.month < 0) {
                    this.month = 11;
                    this.year--;
                }
                this.initDays();
            }
        },
        markDay() {
            this.days.forEach(week => {
                week.forEach(day => {
                    if (day.className.disabled) {
                        return true;
                    }
                    if (this.evnets.some(item => item == day.day)) {
                        day.className.event = true;
                    }
                });
            });
        },
        resize() {
            window.onresize = () => {
                this.setHeight();
            }
        },
        setHeight() {
            this.height = document.getElementsByClassName('day')[0].clientWidth;
        }
    },
    created() {
        this.initCalendar();
        this.resize();
    },
    mounted() {
        this.markDay();
        this.setHeight();
    },
    watch: {
        evnets(val) {
            this.markDay();
        }
    },
}
</script>

<style lang="scss" scoped>
.calendar {
    width: 100%;
    color: #444;
    font-size: 0.9rem;

    &-header {
        overflow: hidden;
        margin-bottom: 16px;
        padding: 16px;

        .month {
            font-size: 16px;
            float: left;
        }

        .operation {
            float: right;

            .btn {
                width: 20px;
                height: 20px;
                background-size: contain;
                display: inline-block;
                cursor: pointer;
                background-position: center center;
            }

            .back {
                width: 36px;
                height: 20px;
                line-height: 20px;
                text-align: center;
                display: inline-block;
                margin: 0 16px;
                vertical-align: top;
            }

            .arrow-left {
                background-image: url('./assets/left-arrow.png');
            }

            .arrow-right {
                background-image: url('./assets/right-arrow.png');
            }
        }
    }

    .weeks {
        display: flex;
        justify-content: center;
        margin-bottom: 16px;

        .week {
            flex: 1;
            text-align: center;
            font-weight: bold;
        }
    }

    .days {
        .a-week {
            display: flex;
            justify-content: center;

            .day {
                flex: 1;
                text-align: center;
                position: relative;
                border: 1px solid #fff;
                margin: 1px;
            }

            .day:hover {
                border-color: #f6f6f6;
                transition: all 0.3s ease-in-out;
            }

            .day.disabled {
                color: #999;
            }

            .day.selected {
                border-color: #91d5ff;
                transition: all 0.3s ease-in-out;
            }

            .day.active {
                background: #e6f7ff;
                border-color: #91d5ff;
            }

            .day.event::after {
                content: '';
                position: absolute;
                bottom: 0px;
                right: 0px;
                width: 0;
                height: 0;
                border-color: transparent transparent #1890ff;
                border-width: 0 0 8px 8px;
                border-style: solid;
            }
        }

    }
}
</style>
