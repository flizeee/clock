<template>
  <div class="menu" v-if="isMenuShown">
    <div class="wrapper">
      <button v-for="v, k in presets" :key="v" @click="schedule = v; refreshSchedule();">載入{{ k }}</button>
    </div>

    <!-- <label>考程</label> -->
    <div class="wrapper">
      <div v-for="x in schedule" :key="x.id">
        <span style="margin-right: 4px;">{{ x.name }}</span>
        <span style="margin-right: 4px;">{{ x.start }} ~ {{ x.end }}</span>
        <button @click="editEvent(schedule.indexOf(x))">編輯</button>
        <button @click="deleteEvent(schedule.indexOf(x))">刪除</button>
      </div>
    </div>

    <!-- <label>新增</label> -->
    <div class="wrapper">
      <select v-model="selectedSubject">
        <option v-for="x in subjects" :value="x" :key="x">{{ x }}</option>
      </select>
      <VueDatePicker class="time-picker" v-model="selectedStart" time-picker />
      <VueDatePicker class="time-picker" v-model="selectedEnd" time-picker />
      <button @click="pushNewEvent">新增</button>
      <br>
    </div>

    <!-- <label>人數</label> -->
    <div class="wrapper">
      <button @click="showPersons = !showPersons;">{{ showPersons ? "隱藏" : "顯示" }}</button>
      <br>
      應到 {{ totalPersons }} 人
      <button @click="totalPersons++">+</button>
      <button @click="totalPersons--">-</button>
      <br>
      實到 {{ totalPersons - absentPersons }} 人
      <button @click="absentPersons--">+</button>
      <button @click="absentPersons++">-</button>
    </div>

    <img class="menu-btn" src="./assets/close-icon.svg" @click="isMenuShown = false" />
  </div>

  <div class="container">
    <div class="center">
      <span id="clock">{{ time }}</span>
    </div>

    <div class="current-event" v-if="currentEvent != null">
      <div class="countdown">
        <span v-if="countdown != null">剩下 {{ countdown }} 分鐘</span>
        <!-- <span v-else-if="countdown == 0">剩下不到 1 分鐘</span> -->
      </div>
      <div class="event-details">
        <span style="margin-right: 20px;">{{ currentEvent.name }}</span>
        <span>{{ currentEvent.start }} ~ {{ currentEvent.end }}</span>
      </div>
    </div>

    <div class="schedule center" v-for="x in activeSchedule" :key="x.id" v-else>
      <span style="margin-right: 20px;">{{ x.name }}</span>
      <span>{{ x.start }} ~ {{ x.end }}</span>
    </div>

    <span class="other" v-if="showPersons">
      應到 {{ totalPersons }} 人
      <br>
      實到 {{ totalPersons - absentPersons }} 人
    </span>

    <img class="menu-btn" src="./assets/menu-icon.svg" @click="isMenuShown = !isMenuShown" v-if="!isMenuShown" />
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    let date = new Date();

    return {
      time: "00:00:00",
      countdown: null,
      scheduleId: 10,
      schedule: [
        { id: 1, name: "英文", start: "14:50", end: "15:00" },
        { id: 2, name: "數學", start: "16:50", end: "17:40" },
        { id: 3, name: "國文", start: "17:50", end: "21:50" }
      ],
      activeSchedule: [],
      currentEvent: null,
      isMenuShown: false,

      selectedSubject: "國文",
      selectedStart: {
        hours: date.getHours(),
        minutes: date.getMinutes()
      },
      selectedEnd: {
        hours: date.getHours(),
        minutes: date.getMinutes()
      },

      subjects: ["國文", "英文", "數學", "電學", "作文", "專一", "專二"],

      showPersons: true,
      totalPersons: 36,
      absentPersons: 0,

      presets: {
        "模考": [
          { id: 1, name: "國文", start: "08:20", end: "10:00" },
          { id: 2, name: "英文", start: "10:20", end: "12:00" },
          { id: 3, name: "專二", start: "13:10", end: "14:50" },
        ],
        "模考D2": [
          { id: 1, name: "數學", start: "08:20", end: "09:40" },
          { id: 2, name: "專一", start: "10:20", end: "12:00" }
        ],
        // "段考": [
        //   { id: 1, name: "英文", start: "14:50", end: "15:00" },
        //   { id: 2, name: "數學", start: "16:50", end: "17:40" },
        //   { id: 3, name: "國文", start: "17:50", end: "21:50" }
        // ]
      },
    };
  },
  mounted() {
    setInterval(() => {
      let date = new Date();
      let h = date.getHours();
      let m = date.getMinutes();
      let s = date.getSeconds();

      let hx = (h < 10) ? "0" + h : h;
      let mx = (m < 10) ? "0" + m : m;
      let sx = (s < 10) ? "0" + s : s;

      this.time = hx + ":" + mx + ":" + sx;

      if (this.currentEvent) {
        let start = this.getDateObject(this.currentEvent.start);
        let end = this.getDateObject(this.currentEvent.end);
        if (date > start && date <= end) {
          let h2 = end.getHours() - h;
          let m2 = end.getMinutes() - m;
          this.countdown = (h2 * 60) + m2 - 1;
        }
        else {
          this.refreshSchedule();
        }
      }
      else {
        for (let x of this.schedule) {
          let start = this.getDateObject(x.start);
          let end = this.getDateObject(x.end);
          if (date > start && date < end) {
            this.currentEvent = x;
          }
        }
      }
    }, 1000);

    this.refreshSchedule();
  },
  watch: {
    schedule() {
      this.refreshSchedule();
    }
  },
  methods: {
    getDateObject(timeString) {
      let date = new Date();
      let data = timeString.split(":");
      date.setHours(parseInt(data[0]), parseInt(data[1]));
      return date;
    },
    getDateString(dateObject) {
      return dateObject.hours.toString().padStart(2, "0") + ":" + dateObject.minutes.toString().padStart(2, "0");
    },
    refreshSchedule() {
      let date = new Date();
      this.activeSchedule = this.schedule.filter((x) => date < this.getDateObject(x.end));
      this.currentEvent = null;
      this.countdown = null;
    },
    editEvent(index) {
      let data = this.schedule.splice(index, 1)[0];
      let start = this.getDateObject(data.start);
      let end = this.getDateObject(data.end);

      this.selectedStart.hours = start.getHours();
      this.selectedStart.minutes = start.getMinutes();

      this.selectedEnd.hours = end.getHours();
      this.selectedEnd.minutes = end.getMinutes();

      this.selectedSubject = data.name;
      
      this.refreshSchedule();
    },
    deleteEvent(index) {
      this.schedule.splice(index, 1);
      this.refreshSchedule();
    },
    pushNewEvent() {
      this.schedule.push({
        id: ++this.scheduleId,
        name: this.selectedSubject,
        start: this.getDateString(this.selectedStart),
        end: this.getDateString(this.selectedEnd)
      });
      this.refreshSchedule();
    }
  }
}
</script>

<style>
body {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  height: 100vh;
  margin: 0;
  user-select: none;
}

.center {
  display: flex;
  align-items: center;
  justify-content: center;
}

#clock {
  font-size: 16vw;
  font-family: monospace;
  font-weight: bolder;
}

.current-event {
  font-weight: bolder;
}

.current-event div {
  width: 100%;
  text-align: center;
}

.current-event .event-details {
  position: absolute;
  top: 30px;
  left: 0;
  font-size: 5vw;
}

.current-event .countdown {
  font-size: 8vw;
}

.schedule {
  font-size: 5vw;
  font-weight: bolder;
}

.other {
  position: absolute;
  right: 10px;
  bottom: 10px;
  font-size: 2vw;
  font-weight: bolder;
}

.menu-btn {
  width: 40px;
  height: 40px;
}

.container .menu-btn {
  position: absolute;
  left: 10px;
  bottom: 10px;
}

div.menu {
  position: absolute;
  bottom: 10px;
  left: 10px;
}

.time-picker {
  width: 150px !important;
}

.menu button {
  margin: 0 2px;
}

.menu .wrapper {
  border: 1px solid grey;
  border-radius: 10px;
  padding: 8px;
  margin: 8px 0;
}

.menu label {
  font-size: large;
  font-weight: bold;
  margin-left: 8px;
}
</style>
