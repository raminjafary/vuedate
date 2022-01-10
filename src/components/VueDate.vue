<template>
  <div class="container">
    <button @click="reset">reset</button>
    <button
      :disabled="disableBack(startMonthDate)"
      :class="disableBack(startMonthDate)"
      @click="updateCalendar(-1)"
    >
      prev
    </button>
    <button @click="updateCalendar(1)">next</button>
    <div class="week-days">
      <div v-for="(w, i) in weekdays" :key="i" class="week-days-item">
        {{ w }}
      </div>
    </div>
    <!-- <template v-if="startMonthDate">
      {{ months[startMonthDate.getMonth()] }} {{ startMonthDate.getFullYear() }}
    </template> -->

    <div class="calendar">
      <template v-for="(week, i) in startMonths">
        <div class="dw" :key="'dw' + i" :id="week[i]">
          <div
            ref="day"
            :key="idx"
            :id="startDay && startDay"
            :class="[
              'day',
              !startDay ? 'not-in-month' : '',
              dayStatus(startDay),
            ]"
            :draggable="true"
            v-for="(startDay, idx) in week"
            @dragstart="onDragStart"
            @dragend="onDragEnd"
            @dragover="onDragOver"
            @dragleave="onDragLeave"
            @dragenter="onDragEnter"
            @drop="onDrop"
            @mouseover="onMouseOver"
            @mousedown="onMouseDown"
            @click="onDayClick(startDay)"
          >
            <template v-if="startDay">
              {{ startDay.getDate() }}
            </template>
          </div>
        </div>
      </template>
    </div>
  </div>
</template>
<script lang="ts">
import Vue from "vue";

export default Vue.extend({
  props: {
    format: {
      type: String,
      default: "YYYY/MM/DD",
    },
    startDate: {
      type: [String, Date],
      default: undefined,
    },
    endDate: {
      type: [String, Date],
      default: undefined,
    },
    minDate: {
      type: [String, Date],
      default: () =>
        new Date(
          new Date().getFullYear(),
          new Date().getMonth(),
          new Date().getDate(),
          0,
          0,
          0
        ),
    },
    maxDate: {
      type: [String, Date],
      default: "",
    },
    minNight: {
      type: Number,
      default: undefined,
    },
    maxNight: {
      type: Number,
      default: undefined,
    },
    selectForward: {
      type: Boolean,
      default: true,
    },
    disabledDates: {
      type: Array,
      default: () => [],
    },
    weekdays: {
      type: Array,
      default: () => ["Sun.", "Mon.", "Tue.", "Wen.", "Thu.", "Fri.", "Sat."],
    },
    months: {
      type: Array,
      default: () => [
        "Jan.",
        "Feb.",
        "Mar.",
        "Apr.",
        "May.",
        "Jun.",
        "Jul.",
        "Aug.",
        "Sep.",
        "Oct",
        "Nov.",
        "Dec.",
      ],
    },
  },
  data() {
    return {
      selectStartDate: new Date(
        new Date().getFullYear(),
        new Date().getMonth(),
        20
      ) as unknown as Date,
      selectEndDate: "" as unknown as Date,
      startMonthDate: "" as unknown as Date,
      endMonthDate: "" as unknown as Date,
      startMonths: [] as unknown as Date[][],
      endMonths: [] as unknown as Date[][],
      count: 0,
      selectMinDate: "" as unknown as Date,
      selectMaxDate: "" as unknown as Date,
      advancedMonths: [],
      dragStartDate: false,
      dragEndDate: false,
      rows: 1,
      startCellIndex: 0,
      startRowIndex: 0,
      moving: null as unknown as HTMLElement,
    };
  },
  created() {
    if (this.minDate) {
      this.selectMinDate = new Date(
        typeof this.minDate === "string" ? this.minDate : this.minDate.getTime()
      );
    }
    if (this.maxDate) {
      this.selectMaxDate = new Date(
        typeof this.maxDate === "string"
          ? this.maxDate
          : (this.maxDate as Date).getTime()
      );
    }

    if (this.maxDate) {
      this.selectMaxDate = new Date(
        typeof this.maxDate === "string"
          ? this.maxDate
          : (this.maxDate as Date).getTime()
      );
    }

    if (this.startDate) {
      const start =
        typeof this.startDate === "string"
          ? this.startDate
          : (this.startDate as Date).getTime();

      this.selectStartDate = new Date(start);
      if (
        this.selectMinDate &&
        this.selectMinDate.getTime() > this.selectStartDate.getTime()
      ) {
        this.selectMinDate = new Date(start);
      }

      if (!this.endDate) {
        this.selectEndDate = new Date(
          this.selectStartDate.getTime() + 24 * 60 * 60 * 100
        );
      } else {
        this.selectEndDate = new Date(
          typeof this.endDate === "string"
            ? this.endDate
            : (this.endDate as Date).getTime()
        );
      }
    }
    this.updateCalendar();
  },
  computed: {},
  methods: {
    createCalendar(
      year = new Date().getFullYear(),
      fromMonth = new Date().getMonth(),
      config = {
        showPreviousMonthDate: false,
        showNextMonthDate: false,
      }
    ) {
      const showPreviousMonthDate = config.showPreviousMonthDate || false;
      const showNextMonthDate = config.showNextMonthDate || false;
      let base = new Date(year, fromMonth, 1, 0, 0, 0);
      let time = new Date(year, fromMonth, 1, 0, 0, 0);

      const months: (Date | any)[][] = [];
      let weeks: (Date | boolean)[] = [];

      let nextMonth = false;
      let completed = false;

      while (!nextMonth || (nextMonth && !completed)) {
        let day = time.getDay();
        let date = time.getDate();
        let month = time.getMonth();

        if (month !== fromMonth) {
          nextMonth = true;
          if (day === 6 || (date === 1 && day === 0)) {
            completed = true;
          }
        }

        if (!nextMonth) {
          weeks[day] = new Date(time.getTime());
        } else {
          weeks[day] = showNextMonthDate ? new Date(time.getTime()) : false;
        }

        if (time.getTime() === base.getTime() && day !== 0) {
          let prevDay = day;
          let prevTime = new Date(time.getTime());
          prevTime.setDate(prevTime.getDate());

          if (showPreviousMonthDate) {
            while (prevDay !== 0) {
              let previousDateTime = new Date(prevTime.getTime());
              prevDay = previousDateTime.getDay();
              weeks[prevDay] = previousDateTime;
              prevTime.setDate(prevTime.getDate() - 1);
            }
          }
        }

        if (
          (time.getTime() === base.getTime() && weeks.length === 7) ||
          (time.getTime() > base.getTime() && day === 6)
        ) {
          months.push(weeks);
          weeks = [];
        }
        time.setDate(time.getDate() + 1);
      }
      return months;
    },
    onMouseDown(e: Event & { target: HTMLDivElement }) {
      this.moving = e.target;

      const cells =
        (e.target.parentNode?.children as unknown as HTMLDivElement[]) || [];

      const rows =
        (e.target.parentNode?.parentNode
          ?.children as unknown as HTMLDivElement[]) || [];

      this.startRowIndex = [...rows].findIndex(
        (el) => el.id === e.target.parentElement?.id
      );

      this.startCellIndex = [...cells].findIndex((el) => el.id === e.target.id);
    },
    onMouseOver(e: Event & { target: HTMLDivElement }) {
      if (
        !this.selectStartDate ||
        (this.selectStartDate && this.selectEndDate)
      ) {
        return;
      }

      Array.prototype.forEach.call(
        document.querySelectorAll(".in-date-range"),
        (el: HTMLElement) => {
          el.classList.remove("in-date-range");
        }
      );

      this.selectTo(e.target);
    },
    selectTo(cell: HTMLElement) {
      const parent = cell.parentNode?.parentNode;
      const rows = cell.parentNode?.parentNode?.children?.length || 0;

      for (let i = this.startRowIndex; i <= rows; i++) {
        let rowCell = parent?.children[i]?.children || [];
        for (let j = 0; j <= rowCell.length; j++) {
          if (
            this.selectStartDate.getTime() <
              new Date(rowCell?.[j]?.id).getTime() &&
            new Date(rowCell?.[j]?.id).getTime() <= new Date(cell.id).getTime()
          ) {
            rowCell?.[j]?.classList.add("in-date-range");
          }
        }
      }
    },
    disableBack(datetime: string | Date) {
      const now = new Date();
      const today = new Date(
        now.getFullYear(),
        now.getMonth(),
        now.getDate(),
        0,
        0,
        0
      );
      datetime = typeof datetime === "string" ? new Date(datetime) : datetime;
      if (datetime && this.selectForward) {
        if (this.selectMinDate) {
          if (datetime.getFullYear() < this.selectMinDate.getFullYear()) {
            return "disabled";
          }
          if (
            datetime.getFullYear() === this.selectMinDate.getFullYear() &&
            datetime.getMonth() <= this.selectMinDate.getMonth()
          ) {
            return "disabled";
          }
        }
      } else {
        if (
          datetime.getFullYear() === today.getFullYear() &&
          datetime.getMonth() <= today.getMonth()
        ) {
          return "disabled";
        }
      }
    },
    reset() {
      this.selectStartDate = "" as unknown as Date;
      this.selectEndDate = "" as unknown as Date;
      this.$emit("reset");
    },
    onDragOver(e: DragEvent & { target: HTMLDivElement }) {
      e.preventDefault();
      if (
        e.target.classList.contains("disabled") ||
        e.target.classList.contains("forbidden") ||
        e.target.classList.contains("not-in-month")
      ) {
        return;
      }
      e.target.style.border = "1px dotted black";
    },
    onDragEnter(e: DragEvent & { target: HTMLDivElement }) {
      e.preventDefault();

      if (
        !this.selectStartDate ||
        (this.selectStartDate && this.selectEndDate)
      ) {
        return;
      }

      Array.prototype.forEach.call(
        document.querySelectorAll(".in-date-range"),
        (el: HTMLElement) => {
          el.classList.remove("in-date-range");
        }
      );

      this.selectTo(e.target);
    },
    onDragLeave(e: DragEvent & { target: HTMLDivElement }) {
      e.preventDefault();
      e.target.style.border = "none";
    },
    onDragEnd(e: DragEvent & { target: HTMLDivElement }) {
      e.target.style.opacity = "1";

      Array.prototype.forEach.call(
        document.querySelectorAll(".day"),
        (el: HTMLElement) => {
          el.style.border = "1px dotted transparent";
        }
      );
    },
    onDragStart(e: DragEvent & { target: HTMLDivElement }) {
      if (
        e.target.classList.contains("start-date") ||
        e.target.classList.contains("end-date")
      ) {
        e.target.style.opacity = "0.4";
        e.dataTransfer?.setData("text", e.target?.id);
        this.dragStartDate = !!e.target.classList.contains("start-date");
        this.dragEndDate = !!e.target.classList.contains("end-date");
      } else {
        e.preventDefault();
        e.stopImmediatePropagation();
        e.stopPropagation();
        return;
      }
    },
    formatDate(datetime: string | Date) {
      if (datetime) {
        datetime = typeof datetime === "string" ? new Date(datetime) : datetime;

        const yyyy = datetime.getFullYear();
        const mm =
          datetime.getMonth() + 1 > 9
            ? datetime.getMonth() + 1
            : `0${datetime.getMonth() + 1}`;
        const dd =
          datetime.getDate() > 9
            ? datetime.getDate()
            : `0${datetime.getDate()}`;
        return this.format
          .replace("YYYY", yyyy.toString())
          .replace("MM", mm.toString())
          .replace("DD", dd.toString());
      }
      return undefined;
    },
    onDrop(e: Event & { target: HTMLDivElement }) {
      e.preventDefault();
      e.stopPropagation();

      const rows =
        (e.target as HTMLDivElement).parentNode?.parentNode?.children || [];
      // const cols = (e.target as HTMLDivElement).parentNode?.children || [];

      for (const row of rows) {
        for (const item of row.children) {
          if (
            item.classList.contains("disabled") ||
            item.classList.contains("forbidden") ||
            item.classList.contains("not-in-month")
          ) {
            break;
          }
          if (this.dragStartDate) {
            if (e.target?.id === item.id) {
              item.classList.add("start-date");
              this.selectStartDate = new Date(e.target.id);
            } else if (
              this.selectEndDate &&
              new Date(item.id).getTime() > new Date(e.target.id).getTime() &&
              new Date(item.id).getTime() < this.selectEndDate.getTime()
            ) {
              item.classList.add("in-date-range");
              item.classList.remove("start-date");
            } else if (
              new Date(item.id).getTime() < new Date(e.target.id).getTime() &&
              item.classList.contains("in-date-range")
            ) {
              item.classList.remove("in-date-range");
            }
          } else {
            if (e.target?.id === item.id) {
              item.classList.add("end-date");
              this.selectEndDate = new Date(e.target.id);
            } else if (
              this.selectEndDate &&
              new Date(item.id).getTime() < new Date(e.target.id).getTime() &&
              new Date(item.id).getTime() > this.selectEndDate.getTime()
            ) {
              item.classList.add("in-date-range");
              item.classList.remove("start-date");
            } else if (
              new Date(item.id).getTime() > new Date(e.target.id).getTime() &&
              item.classList.contains("in-date-range")
            ) {
              item.classList.remove("in-date-range");
            }
          }
        }
      }
      this.onDayClick(new Date(e.target.id));
    },
    onDayClick(startDay: Date) {
      if (startDay) {
        if (!this.selectStartDate) {
          this.selectStartDate = startDay;
        } else if (!this.selectEndDate) {
          if (
            this.selectStartDate &&
            startDay.getTime() < this.selectStartDate.getTime()
          ) {
            this.selectEndDate = this.selectStartDate;
            this.selectStartDate = startDay;
          } else {
            this.selectEndDate = startDay;
          }
        } else if (startDay.getTime() < this.selectStartDate.getTime()) {
          this.selectStartDate = startDay;
        } else if (startDay.getTime() > this.selectEndDate.getTime()) {
          this.selectEndDate = startDay;
        } else if (
          startDay.getTime() > this.selectStartDate.getTime() &&
          startDay.getTime() < this.selectEndDate.getTime()
        ) {
          if (this.count % 2 === 0) {
            this.selectStartDate = startDay;
          } else {
            this.selectEndDate = startDay;
          }
          this.count++;
        }

        if (this.selectStartDate && this.selectEndDate && this.minNight) {
          const limitDate =
            this.selectStartDate.getTime() +
            this.minNight * 1000 * 60 * 60 * 24;
          if (this.selectEndDate.getTime() < limitDate) {
            this.selectEndDate = new Date(limitDate);
          }
        }

        if (this.selectStartDate && this.selectEndDate && this.maxNight) {
          const limitDate =
            this.selectStartDate.getTime() +
            this.maxNight * 1000 * 60 * 60 * 24;
          if (this.selectEndDate.getTime() > limitDate) {
            this.selectEndDate = new Date(limitDate);
          }
        }

        this.$emit("update", {
          start: this.formatDate(this.selectStartDate),
          end: this.formatDate(this.selectEndDate),
        });
      }
    },
    updateCalendar(offset = 0) {
      if (!this.startMonthDate) {
        this.startMonthDate = this.selectStartDate
          ? new Date(this.selectStartDate.getTime())
          : new Date(new Date().getFullYear(), new Date().getMonth());
      }

      this.startMonthDate.setMonth(this.startMonthDate.getMonth() + offset);
      this.endMonthDate = new Date(
        this.startMonthDate.getFullYear(),
        this.startMonthDate.getMonth() + 1
      );

      // this.startMonths = [];
      this.endMonths = [];

      this.startMonths = this.startMonths.concat(
        this.createCalendar(
          this.startMonthDate.getFullYear(),
          this.startMonthDate.getMonth()
        )
      );

      this.rows++;

      this.endMonths = this.createCalendar(
        this.endMonthDate.getFullYear(),
        this.endMonthDate.getMonth()
      );
    },
    dayStatus(startDay: Date) {
      const classList = [];

      if (startDay) {
        const now = new Date();

        if (this.selectMinDate.getTime() > startDay.getTime()) {
          classList.push("disabled");
        } else if (
          this.selectMaxDate &&
          this.selectMaxDate.getTime() < startDay.getTime()
        ) {
          classList.push("disabled");
        } else if (this.disabledDates.indexOf(this.formatDate(startDay)) > -1) {
          classList.push("disabled");
          classList.push("forbidden");
        } else if (
          this.selectStartDate &&
          this.selectStartDate.getTime() > startDay.getTime() &&
          !this.selectForward
        ) {
          classList.push("disabled");
        } else if (
          this.selectStartDate &&
          this.selectStartDate.getTime() === startDay.getTime()
        ) {
          classList.push("start-date");
        } else if (
          this.selectEndDate &&
          this.selectEndDate.getTime() === startDay.getTime()
        ) {
          classList.push("end-date");
        } else if (
          this.selectStartDate &&
          this.selectEndDate &&
          startDay.getTime() > this.selectStartDate.getTime() &&
          startDay.getTime() < this.selectEndDate.getTime()
        ) {
          classList.push("in-date-range");
        }

        if (
          this.selectStartDate &&
          ((Number.isInteger(this.minNight) && this.minNight > 0) ||
            (Number.isInteger(this.maxNight) && this.maxNight > 0))
        ) {
          const night =
            Math.abs(startDay.getTime() - this.selectStartDate.getTime()) /
            (1000 * 60 * 60 * 24);
          if (night < this.minNight) {
            classList.push("disabled");
          } else if (night > this.maxNight) {
            classList.push("disabled");
          }
        }

        if (
          now.getFullYear() === startDay.getFullYear() &&
          now.getMonth() === startDay.getMonth() &&
          now.getDate() === startDay.getDate()
        ) {
          classList.push("today");
        }
      }
      return classList;
    },
  },
});
</script>
<style scoped>
* {
  box-sizing: border-box;
}
.container {
  display: inline-block;
  vertical-align: top;
  width: 280px;
}
.week-days {
  width: 100%;
  height: 32px;
}

.week-days-item {
  float: left;
  font-size: 12px;
  font-weight: 500;
  width: calc(100% / 7);
  height: 20px;
  color: #505050;
  text-align: center;
  line-height: 20px;
}

.dw {
  display: flex;
  width: 100%;
  height: 40px;
}

.day {
  width: calc(100% / 7);
  float: left;
  position: relative;
  width: 40px;
  border: 1px dotted transparent;
  height: 40px;
  font-size: 16px;
  font-weight: 500;
  line-height: 40px;
  color: #505050;
  background-color: transparent;
  text-align: center;
  cursor: pointer;
  transition: background-color 0.4s cubic-bezier(0.165, 0.84, 0.44, 1);
}
/* .day::before,
.day::after {
  content: "";
  position: absolute;
  width: 0px;
  height: 100%;
  left: 0;
  background-color: transparent;
  opacity: 0;
  transition: opacity 0.4s cubic-bezier(0.165, 0.84, 0.44, 1),
    background-color 0.4s cubic-bezier(0.165, 0.84, 0.44, 1),
    width 0.4s cubic-bezier(0.165, 0.84, 0.44, 1);
}
.day::after {
  left: auto;
  right: 0;
} */
.day.disabled {
  color: #ececec;
  pointer-events: none;
}
.day.in-date-range {
  background: linear-gradient(
    90deg,
    rgba(60, 150, 226, 0.1) 0%,
    rgba(96, 96, 96, 0.1) 20%,
    rgba(60, 150, 226, 0.1) 60%,
    rgba(96, 96, 96, 0.1) 100%
  );
}
.day.start-date {
  position: relative;
  background-color: #3c96e2;
  border-top-left-radius: 12px;
  border-bottom-left-radius: 12px;
  color: white !important;
}

.day.end-date {
  position: relative;
  background-color: #606060;
  border-top-right-radius: 12px;
  border-bottom-right-radius: 12px;
  color: white;
}

.day.today {
  color: red;
}
.day.forbidden {
  color: #fed9d8;
  cursor: not-allowed;
}

.day.end-date:after {
  content: url("../assets/black-handle.svg");
  position: absolute;
  background-size: 50px;
  opacity: 0;
  top: 50%;
  right: -5px;
  animation: shake 0.82s cubic-bezier(0.36, 0.07, 0.19, 0.97) both;
  transform: translate3d(0, -14px, 0);
  backface-visibility: hidden;
  perspective: 1000px;
  opacity: 1;
}

.day.start-date:after {
  content: url("../assets/blue-handle.svg");
  position: absolute;
  background-size: 50px;
  opacity: 0;
  top: 50%;
  left: -5px;
  animation: shake 0.82s cubic-bezier(0.36, 0.07, 0.19, 0.97) both;
  transform: translate3d(0, -14px, 0);
  backface-visibility: hidden;
  perspective: 1000px;
  opacity: 1;
}

@keyframes shake {
  10%,
  90% {
    transform: translate3d(-1px, -14px, 0);
  }

  20%,
  80% {
    transform: translate3d(2px, -14px, 0);
  }

  30%,
  50%,
  70% {
    transform: translate3d(-4px, -14px, 0);
  }

  40%,
  60% {
    transform: translate3d(4px, -14px, 0);
  }
}

/* @media(hover: hover) and (pointer: fine) {
    .day:hover {
        color: red;
    }
} */
</style>
