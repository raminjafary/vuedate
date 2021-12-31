<template>
  <div class="container">
    <div class="week-days">
      <div v-for="(w, i) in weekdays" :key="i" class="week-days-item">
        {{ w }}
      </div>
    </div>

    <div v-for="(week, i) in startDate" :key="i" class="dw">
      <!-- {{week[i] && months[week[i].getMonth()] + ' ' + week[i].getFullYear()}} -->
      <div
        class="day"
        v-for="(startDay, idx) in week"
        :key="idx"
        :id="startDay && startDay"
        :class="dayStatus(startDay)"
        @dragstart="onDragStart"
        @dragover="($event) => $event.preventDefault()"
        @drop="onDrop"
        @click="onDayClick(startDay)"
        :draggable="true"
        ref="day"
      >
        <span v-if="startDay"> {{ startDay.getDate() }} </span>
      </div>
    </div>
  </div>
</template>
<script lang="ts">
import Vue from "vue";

export default Vue.extend({
  props: {
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
      startDate: [] as Date[][],
      selectStartDate: "" as unknown as Date,
      selectEndDate: "" as unknown as Date,
      count: 0,
    };
  },
  mounted() {
    this.startDate = this.createCalendar();
  },
  computed: {},
  methods: {
    createCalendar(
      year = new Date().getFullYear(),
      fromMonth = new Date().getMonth(),
      config = {}
    ) {
      let base = new Date(year, fromMonth, 1, 0, 0, 0);
      let time = new Date(year, fromMonth, 1, 0, 0, 0);

      const months = [];
      let weeks = [];

      let nextMonth = false;
      let completed = false;

      while (!nextMonth || (!nextMonth && !completed)) {
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
        }

        if (time.getTime() === base.getTime() && day !== 0) {
          let prevDay = day;
          let prevTime = new Date(time.getTime());
          prevTime.setDate(prevTime.getDate());
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
    onDragStart(e: Event) {
      if (e.target.classList.contains("start-date")) {
        e.dataTransfer?.setData("text", e.target?.id);
      }
    },
    onDrop(e: Event) {
      e.preventDefault();
      // this.selectStartDate = new Date(e.dataTransfer?.getData("text"));
      this.onDayClick(
        new Date(e.dataTransfer?.getData("text")) as unknown as Date
      );
      const classes = this.dayStatus(
        new Date(e.dataTransfer?.getData("text")) as unknown as Date
      );
      console.log(e.target?.classList.add(classes.join("")));
      console.log(classes);
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
          if (this.count) {
            this.selectStartDate = startDay;
          } else {
            this.selectEndDate = startDay;
          }
          this.count++;
        }

        this.$emit("update", {
          start: this.selectStartDate,
          end: this.selectEndDate,
        });

        // if (this.selectStartDate && this.selectEndDate) {
        //   this.updateValue();
        // }
      }
    },
    // updateValue() {},
    dayStatus(startDay: Date) {
      const classList = [];

      if (startDay) {
        const now = new Date();

        if (
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
  display: block;
  width: 100%;
  height: 40px;
}

.day {
  width: calc(100% / 7);
  float: left;
  position: relative;
  width: 40px;
  height: 40px;
  font-size: 16px;
  font-weight: 500;
  line-height: 40px;
  color: #505050;
  border: solid 1px #b2d7ff;
  background-color: transparent;
  text-align: center;
  cursor: pointer;
  transition: background-color 0.4s cubic-bezier(0.165, 0.84, 0.44, 1);
}
.day::before,
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
}
.day.disabled {
  color: #ececec;
  pointer-events: none;
}
.day.in-date-range {
  background-color: #b2d7ff;
}
.day.start-date {
  position: relative;
  background-color: #b2d7ff;
}
.day.start-date .day::before {
  width: 4px;
  background-color: #0088ff;
  opacity: 1;
}
.day.end-date {
  position: relative;
  background-color: #b2d7ff;
}

.day.end-date .day::after {
  width: 4px;
  background-color: #0088ff;
  opacity: 1;
  transition: opacity 0.2s cubic-bezier(0.165, 0.84, 0.44, 1),
    background-color 0.2s cubic-bezier(0.165, 0.84, 0.44, 1),
    width 0.2s cubic-bezier(0.165, 0.84, 0.44, 1);
}
.day.today {
  border: solid 1px #0088ff;
}
</style>
