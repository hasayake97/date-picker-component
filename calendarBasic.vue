<template>
    <div class="el-date-picker__views-main">
        <!-- 起始区域 -->
        <div class="el-date-picker-main-start">
            <div class="el-date-picker-main-title">{{ title }}</div>
            <div class="el-date-picker-main-middle">
                <span class="el-date-picker-date">
                    {{ `${this.ymd.year}年${this.ymd.month}月` }}
                </span>
                <span class="el-date-picker-control-area" @click.stop="controlHandler($event)">
                    <i class="icon-shangyinian" data-mark="preYear"></i>
                    <i class="icon-shangyigeyue" data-mark="preMonth"></i>
                    <i class="icon-jintian" data-mark="today"></i>
                    <i class="icon-xiayigeyue" data-mark="nextMonth"></i>
                    <i class="icon-xiayinian" data-mark="nextYear"></i>
                </span>
            </div>
            <!-- 日历区域 -->
            <div class="el-date-picker-calender__wrapper">
                <!-- 日历头部 -->
                <table>
                    <thead class="el-date-picker-header">
                        <tr>
                            <th
                                v-for="i in calenderInfos.headerList"
                                :key="i">
                                {{ i }}
                            </th>
                        </tr>
                    </thead>

                    <tbody class="el-date-picker-body">
                        <tr v-for="(i, idx) in dateList" :key="idx" @click.stop="selectedDate($event)">
                            <td
                                v-for="item in i"
                                :key="item.day"
                                :class="[
                                    item.class,
                                ]"
                                >
                                <span
                                    :class="[ isSelected(item.date) ? 'selected' : '']"
                                    :data-date="item.date">
                                    {{ item.day }}
                                </span>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</template>

<script>

export default {
    name: 'calendarBasic',
    props: {
        title: {
            type: String,
            default: ''
        },
        mark: {
            type: String,
            default: ''
        }
    },
    data() {
        return {
            calenderInfos: {
                headerList: ['周日', '周一', '周二', '周三', '周四', '周五', '周六']
            },
            ymd: {
                year: null,
                month: null,
                day: null
            },
            dateList: [],
            isLineScope: true,
            isNewSelected: true
        }
    },
    methods: {
        formatTimeStamp(timeStamp) {
            const date = new Date(timeStamp + 8 * 3600 * 1000)
                .toJSON()
                .substr(0, 10)
                .replace(/-/g, (v, i) => ({
                    4: '年',
                    7: '月'
                }[i]))
            return `${date}日`
        },
        getCurMonthCountDays(date = new Date().getTime()) {
            const curDate = new Date(date)
            curDate.setMonth(curDate.getMonth() + 1)
            curDate.setDate(0)
            return curDate.getDate()
        },
        initYMD(date = new Date().getTime()) {
            const toDay = new Date(date)
            this.ymd = {
                year: toDay.getFullYear(),
                month: toDay.getMonth() + 1,
                day: toDay.getDate()
            }
        },
        addZero(str) {
            return str.replace(/-(\d)(?!\d)/g, '-0$1')
        },
        generateDayList() {
            /**
             * 生成日期列表长度为42
             * 包括当前月，上月，下月
             */
            const maxDay = 42
            const offsetPreMonth = this.currMonthFirstDay(`${this.ymd.year}-${this.ymd.month}`)
            const curMonth = new Array(this.getCurMonthCountDays(`${this.ymd.year}-${this.ymd.month}`))
                .fill().map((item, idx) => ({
                    day: idx + 1,
                    date: this.addZero(`${this.ymd.year}-${this.ymd.month}-${idx + 1}`),
                    class: 'cur-month'
                }))


            let allMonth = []
            let nextArr = []
            if (offsetPreMonth) {
                const preMonth = new Array(this.getCurMonthCountDays(this.checkDateNormal(`${this.ymd.year}-${this.ymd.month - 1}-${this.ymd.day}`)))
                    .fill().map((item, idx) => ({
                        day: idx + 1,
                        date: this.addZero(`${this.ymd.year}-${this.ymd.month - 1}-${idx + 1}`),
                        class: 'pre-month'
                    }))
                const preAndCur = [...preMonth.slice(-offsetPreMonth), ...curMonth]
                nextArr = new Array(maxDay - preAndCur.length).fill().map((i, idx) => ({
                    day: idx + 1,
                    date: this.addZero(`${this.ymd.year}-${this.ymd.month + 1}-${idx + 1}`),
                    class: 'next-month'
                }))
                allMonth = [...preAndCur, ...nextArr]
            } else {
                nextArr = new Array(maxDay - curMonth.length).fill().map((i, idx) => ({
                    day: idx + 1,
                    date: this.addZero(`${this.ymd.year}-${this.ymd.month + 1}-${idx + 1}`),
                    class: 'next-month'
                }))
                allMonth = [...curMonth, ...nextArr]
            }

            let tmpArr = []
            this.dateList = []
            allMonth.map((i, idx) => {
                const index = idx + 1
                tmpArr.push(i)
                if (!(index % 7)) {
                    this.dateList.push(tmpArr)
                    tmpArr = []
                }
                return ''
            })
            this.dateList.push(tmpArr)
        },
        controlHandler(event) {
            const e = event || window.event
            const mark = e.target.getAttribute('data-mark')

            if (mark) {
                this[`${mark}Handler`]()
            }
        },
        currMonthFirstDay(date) {
            return new Date(date).getDay()
        },
        preMonthHandler() {
            this.initYMD(this.checkDateNormal(`${this.ymd.year}-${this.ymd.month - 1}-${this.ymd.day}`))
        },
        preDayHandler() {
            this.initYMD(this.checkDateNormal(`${this.ymd.year}-${this.ymd.month}-${this.ymd.day - 1}`))
        },
        preYearHandler() {
            this.initYMD(this.checkDateNormal(`${this.ymd.year - 1}-${this.ymd.month}-${this.ymd.day}`))
        },
        todayHandler() {
            this.initYMD()
            this.$emit('selectToday', {
                mark: this.mark,
                date: this.addZero(`${this.ymd.year}-${this.ymd.month}-${this.ymd.day}`)
            })
        },
        nextDayHandler() {
            this.initYMD(this.checkDateNormal(`${this.ymd.year}-${this.ymd.month}-${this.ymd.day + 1}`))
        },
        nextMonthHandler() {
            this.initYMD(this.checkDateNormal(`${this.ymd.year}-${this.ymd.month + 1}-${this.ymd.day}`))
        },
        nextYearHandler() {
            this.initYMD(this.checkDateNormal(`${this.ymd.year + 1}-${this.ymd.month}-${this.ymd.day}`))
        },
        // 检验日期是否正常
        checkDateNormal(date) {
            const dateArr = date.split('-').map(i => parseInt(i, 10))
            const oldList = [...dateArr]

            if (dateArr[2] <= 0) {
                dateArr[1] -= 1
                dateArr[2] = this.getCurMonthCountDays(`${oldList[0]}-${oldList[1] - 1}`)
            } else if (dateArr[2] > this.getCurMonthCountDays(`${oldList[0]}-${oldList[1]}`)) {
                dateArr[1] += 1
                dateArr[2] = 1
            }

            if (dateArr[1] <= 0) {
                dateArr[0] -= 1
                dateArr[1] = 12
            } else if (dateArr[1] > 12) {
                dateArr[0] += 1
                dateArr[1] = 1
            }
            return this.addZero(`${dateArr[0]}-${dateArr[1]}-${dateArr[2]}`)
        },
        selectedDate(event) {
            const e = event || window.event
            const el = e.target
            const realEl = el.tagName === 'SPAN' ? el : el.firstElementChild
            realEl.classList.add('selected')
            this.$emit('selectChange', realEl.getAttribute('data-date'))
        },
        isSelected(date) {
            return this.$parent.dateResultArr.includes(date) && this.isNewSelected
        }
    },
    created() {
        const isStartOrEnd = () => {
            const date = new Date()
            if (this.mark === 'end') {
                date.setMonth(date.getMonth() + 1)
            }
            return date.getTime()
        }
        this.initYMD(isStartOrEnd())
    },
    watch: {
        ymd() {
            this.generateDayList()
        },
        '$parent.resultBakArr'(val) {
            this.isNewSelected = !(val.length === 1
                && this.$parent.dateResultArr.length === 2)
        },
        '$parent.show'(val) {
            if (!val && this.$parent.resultBakArr.length === 1) {
                const el = document.querySelector('.selected')
                this.isNewSelected = true
                
                try {
                    if (!this.$parent.dateResultArr.includes(el.getAttribute('data-date'))) {
                        el.classList.remove('selected')
                    }
                } catch (err) { }
            }
        }
    }
}
</script>


<style lang="scss" scoped>
* {
    box-sizing: border-box;
}
.el-date-picker__views-main {
    width: 306px;
    min-height: 240px;
    background-color: #2C3134;
    position: relative;
    padding: 15px 20px 9px;
    & .el-date-picker-main-title {
        color: #8CC4FF;
        font-size: 14px;
    }
    & .el-date-picker-main-middle {
        margin-top: 10px;
        display: flex;
        justify-content: space-between;
        align-items: center;
        & > .el-date-picker-date {
            color: #fff;
            font-size: 12px;
            font-weight: bold;
        }
        &  .el-date-picker-control-area {
            color: #8CC4FF;
            & > i {
                font-size: 12px;
                margin: 0 4px;
                cursor: pointer;
                &:hover {
                    opacity: .5;
                }
            }
        }
    }
    & .el-date-picker-calender__wrapper {
        margin-top: 10px;
        & > table {
            min-width: 100%;
            color: #C0DFFF;
            border-spacing: 0;
            text-align: center;
            th {
                padding: 8px 0;
                box-sizing: border-box;
                border-bottom: 2px solid rgba(151, 151, 151, .5);
            }
            td {
                cursor: pointer;
                // background-color: #666;
                position: relative;
                &:hover {
                    transform: scale(1.2);
                    color: #8CC4FF;
                }
                span {
                    width: 25px;
                    line-height: 25px;
                    display: inline-block;
                    z-index: 1;
                }
            }
            @mixin not-current-month {
                color: #B7B8B9;
                opacity: .3;
                & > span {
                    background: transparent;
                    color: #B7B8B9;
                }
            }
            & .pre-month {
                @include not-current-month;
            }
            & .next-month {
                @include not-current-month;
            }
            & .cur-month {
                color: #B7B8B9;
                opacity: .9;
            }
            & .selected {
                background:rgba(140,196,255,0.2);
                color: #8CC4FF;
                border-radius: 50%;
            }
        }
    }
}
</style>
