<template>
  <div class="schedule" v-for="x in activeSchedule" v-bind:key="x.name">
    <span style="margin-right: 20px;">{{ x.name }}</span>
    <span>{{ x.start }} ~ {{ x.end }}</span>
  </div>
</template>

<script>
export default {
  name: 'MySchedule',
  data() {
    return {
      time: "",
      schedule: [
        {name: "英文", start: "14:50", end: "15:00"},
        {name: "數學", start: "16:50", end: "17:00"},
        {name: "國文", start: "17:50", end: "18:50"}
      ],
      activeSchedule: []
    }
  },
  mounted() {
    setInterval(() => {
      let date = new Date();
      let h = date.getHours();
      let m = date.getMinutes();
      let s = date.getSeconds();
      
      h = (h < 10) ? "0" + h : h;
      m = (m < 10) ? "0" + m : m;
      s = (s < 10) ? "0" + s : s;
      
      this.time = h + ":" + m + ":" + s;
    }, 1000);

    this.refreshSchedule();
  },
  watch: {
    schedule() {
      this.refreshSchedule();
    }
  },
  methods: {
    refreshSchedule() {
      let date = new Date();

      this.activeSchedule = this.schedule.filter((x) => {
        let date2 = new Date();
        let data = x.end.split(":");
        date2.setHours(parseInt(data[0]), parseInt(data[1]), 0);
        return date < date2;
      });
    }
  }
}
</script>

<style>
.schedule {
  display: inline-block;
  font-size: 4vw;
  /* font-family: monospace; */
  font-weight: bolder;
}
</style>
