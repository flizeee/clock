<template>
  <div>
    <span id="clock">{{ time }}</span>
  </div>
  <span class="other">
    應到<br>
    實到
  </span>
</template>

<script>
export default {
  name: 'MyClock',
  data() {
    return {
      time: "00:00:00"
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
  }
}
</script>

<style>
div {
  width: 100%;
}

#clock {
  font-size: 16vw;
  font-family: monospace;
  font-weight: bolder;
}

.other {
  position: absolute;
  right: 10px;
  bottom: 10px;
  font-size: 3vw;
}
</style>
