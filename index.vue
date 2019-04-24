<!--
 * @Description: 日历选择组件
 * @Author: your name
 * @LastEditors: Please set LastEditors
 * @Date: 2019-04-15 10:01:18
 * @LastEditTime: 2019-04-19 16:13:42
 -->

<template>
    <div class="el-date-picker__wrapper" @click.stop="show = true">
        <div class="el-date-picker-selector">
            <span class="el-date-picker-start">起止日期</span>
            <div class="el-date-picker-input__wrapper">
                <input
                    @keyup.enter="datePickerCheck($event, 0)"
                    :value="dateResultArr[0]"
                    class="el-date-picker-input"
                    type="text">
                    <span class="el-date-picker-text">至</span>
                <input
                    @keyup.enter="datePickerCheck($event, 1)"
                    :value="dateResultArr[1]"
                    class="el-date-picker-input"
                    type="text">
            </div>
        </div>
        <transition name="slide">
            <div class="el-date-picker__views" v-show="show">
                <calendar-basic
                    title="起始"
                    mark="start"
                    @selectToday="selectToday"
                    @selectChange="setDateValue">
                </calendar-basic>
                <div class="el-date-picker-middle-line"></div>
                <calendar-basic
                    title="截止"
                    mark="end"
                    @selectToday="selectToday"
                    @selectChange="setDateValue">
                </calendar-basic>
            </div>
        </transition>
    </div>
</template>

<script>
import calendarBasic from './calendarBasic.vue'

export default {
    name: 'date-picker',
    components: {
        calendarBasic
    },
    beforeCreate() {
        document.addEventListener('click', () => {
            this.show = false
        }, false)
    },
    data() {
        return {
            dateResultArr: [],
            resultBakArr: [],
            oldDateResult: [],
            show: false
        }
    },
    methods: {
        setDateValue(date) {
            this.resultBakArr.push(date)
            this.resultBakHandler()
        },
        resultBakHandler() {
            if (this.resultBakArr.length == 2) {
                this.swapData(this.resultBakArr)
                this.show = false
                this.dateResultArr = [ ...this.resultBakArr ]
                this.resultBakArr = []
            }
        },
        selectToday(data) {
            this.resultBakArr.push(data.date)
            if (this.resultBakArr.length <= 1) {
                this.dateResultArr = [ ...this.resultBakArr ]
            }
            this.resultBakHandler()
        },
        datePickerCheck(event, idx) {
            const e = event || window.event
            const reg = /^(\d{4})-(\d{1,2})-(\d{1,2})$/g
            const value = e.target.value
            if (reg.test(value)) {
                this.dateResultArr.splice(idx, 1, this.$children[0]['checkDateNormal'](value))
            } else {
                this.dateResultArr.splice(idx, 1, this.dateResultArr[idx])
            }

            if (this.dateResultArr.length == 2) {
                this.swapData(this.dateResultArr)
            }
        },
        swapData(arr) {
            if (new Date(arr[1]).getTime() < new Date(arr[0]).getTime()) {
                [arr[0], arr[1]] = [arr[1], arr[0]]
            }
        }
    },
    watch: {
        dateResultArr(val) {
            this.$emit('input', val)
        } 
    }
}
</script>

<style lang="scss" scoped>
* {
    box-sizing: border-box;
}
.el-date-picker__wrapper {
    display: inline-block;
    font-size: 12px;
    height: 30px;
    background: #19262E;
    position: relative;
    & > .el-date-picker-selector {
        display: flex;
        align-items: center;
        & > .el-date-picker-start {
            color: #C0DFFF;
        }
        & > .el-date-picker-input__wrapper {
            height: 100%;
            margin-left: 20px;
            padding-right: 48px;
            border: 1px solid #696969;
            border-radius: 2px;
            position: relative;
            color: #fff;
            & > .el-date-picker-input {
                color: inherit;
                width: 100px;
                height: 28px;
                padding: 7px 7px 8px;
                background: #19262E;
                border: none;
                outline: none;
                cursor: pointer;
                text-align: center;
            }
            &::after {
                content: '\e632';
                color: #696969;
                font-family: "iconfont" !important;
                font-size: 16px;
                font-style: normal;
                -webkit-font-smoothing: antialiased;
                -moz-osx-font-smoothing: grayscale;
                position: absolute;
                top: 6px;
                right: 10px;
            }
        }
    }
    .el-date-picker__views {
        position: absolute;
        left: 20px;
        top: 40px;
        max-width: 614px;
        background: #2C3134;
        box-shadow:0px 2px 4px 0px rgba(0,0,0,0.63);
        transform-origin: 0 -15px 0;
        display: flex;
        &::after {
            content: '';
            position: absolute;
            left: calc(50% - 30px);
            top: -25px;
            border: 15px solid #2C3134;
            border-left-color: transparent;
            border-top-color: transparent;
            border-right-color: transparent;
        }
        & > .el-date-picker-middle-line {
            width: 2px;
            height: 200px;
            background: #979797;
            margin-top: 40px;
            opacity: .5;
        }
    }
}
.slide-enter-active {
    transition: transform .18s ease-out,
                opacity .18s ease-out;
}
.slide-leave-active {
    transition: transform .06s ease-out,
                opacity .06s ease-out;
}
.slide-enter-to, .slide-leave {
    transform: scaleY(1);
    opacity: 1;
}
.slide-enter, .slide-leave-to {
    transform: scaleY(0);
    opacity: 0;
}
</style>
