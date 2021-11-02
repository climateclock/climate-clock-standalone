<template>
  <div v-if="ready" id="deadline">
    <div class="header">{{ carbon ? carbon.labels[0] : "" }}</div>
    <div class="side">
      <div v-for="c in 'DEADLINE'.slice()" :key="c">{{ c }}</div>
    </div>
    <div class="group">
      <div class="value">{{ countdown.years }}</div>
      <div class="label">Year{{ countdown.years > 1 ? "s" : " " }}</div>
    </div>
    <div class="group">
      <div class="value">{{ countdown.days }}</div>
      <div class="label">Day{{ countdown.days > 1 ? "s" : " " }}</div>
    </div>
    <div class="group">
      <div class="value">{{ countdown.hours }}</div>
      <div class="label">:</div>
      <div class="value">{{ countdown.minutes }}</div>
      <div class="label">:</div>
      <div class="value">{{ countdown.seconds }}</div>
    </div>
  </div>
  <div v-if="ready" id="lifeline">
    <div class="side">
      <div v-for="c in 'LIFELINE'.slice()" :key="c">{{ c }}</div>
    </div>
    <div class="group">
      <div class="value">{{ renewablePercent }}</div>
      <div class="label">Year{{ countdown.years > 1 ? "s" : " " }}</div>
    </div>
    <div class="group">
      <div class="value">{{ renewablePercent }}</div>
      <div class="label">Year{{ countdown.years > 1 ? "s" : " " }}</div>
    </div>
  </div>
  <div id="branding" @click="handleClick">
    <img src="./climateclock.svg" />
    <div>
      <h1>ClimateClock</h1>
      <h4>©{{ new Date().getFullYear() }} ClimateClock.world</h4>
    </div>
  </div>
</template>

<script>
import { DateTime } from "luxon"

const TICK_INTERVAL = 250

export default {
  data: () => ({
    ready: false,
    now: null,
    deadline: null,

    feed: "",

    newsfeed: {},
    carbon: {},
    renewables: {},
    gcf: {},
    open: window.open,
  }),
  computed: {
    countdown() {
      if (!(this.deadline && this.now)) return {}
      let delta = this.deadline.diff(this.now, [
        "years",
        "days",
        "hours",
        "minutes",
        "seconds",
      ])
      return {
        years: delta.get("years"),
        days: this.pad(delta.get("days"), 3),
        hours: this.pad(delta.get("hours")),
        minutes: this.pad(delta.get("minutes")),
        seconds: this.pad(delta.get("seconds")),
      }
    },
    renewablePercent() {
      let tElapsed = this.now - new Date(this.renewables.timestamp).getTime()
      return (
        this.renewables.initial +
        (tElapsed / 1000) * this.renewables.rate
      ).toFixed(9)
    },
    gcfValue() {
      let tElapsed = this.now - new Date(this.gcf.timestamp).getTime()
      return (this.gcf.initial + (tElapsed / 1000) * this.gcf.rate).toFixed(2)
    },
    indieValue() {
      let tElapsed = this.now - new Date(this.indie.timestamp).getTime()
      return (
        (this.indie.initial + (tElapsed / 1000) * this.indie.rate) *
        1e6
      ).toLocaleString()
    },
    mode() {
      let a = 7,
        b = 13,
        c = 20

      if (!this.now) {
        return false
      } else {
        let x = this.now.second % c
        return x < a ? 0 : x < b ? 1 : 2
      }
    },

    // Items below are skin/theme-specific
    animationDuration() {
      return { animationDuration: 0.15 * this.feedText.length + "s" }
    },
    feedText() {
      return (this.lifeline ? `${this.lifeline} | ` : "") + this.feed
    },
  },
  methods: {
    pad(n, length = 2) {
      return `${"0".repeat(length)}${Math.trunc(n)}`.slice(-length)
    },
    handleClick() {
      if (!window.location.hostname.includes("climateclock.world")) {
        window.open("https://climateclock.world")
      } else {
        this.showChart = !this.showChart
      }
    },
  },
  created() {
    fetch("https://api.climateclock.world/v1/clock?device=widget")
      .then((response) => response.json())
      .then((data) => {
        let modules = data.data.modules
        this.carbon = modules.carbon_deadline_1
        this.renewables = modules.renewables_1
        this.newsfeed = modules.newsfeed_1
        this.gcf = modules.green_climate_fund_1
        this.indie = modules.indigenous_land_1

        this.deadline = DateTime.fromISO(this.carbon.timestamp)
        this.feed =
          this.newsfeed.newsfeed.map((n) => n.headline).join(" | ") + " | "
        this.ready = true
      })
    setInterval(() => {
      this.now = DateTime.now()
    }, TICK_INTERVAL)
  },
  watch: {},
}
</script>
<style lang="scss">
@import "matthewha";
/*
@font-face {
  font-family: "CCKlima";
  src: url("./Klima-Bold.otf") format("opentype");
}
@font-face {
  font-family: "CCKlima";
  src: url("./Klima-Ultra.otf") format("opentype");
  font-weight: bold;
}
 */

$deadline: #eb1c23;
$lifeline: #4aa1cc;

*,
*:before,
*:after {
  box-sizing: border-box;
}
body {
  margin: 0;
  color: white;
  background: black;
}
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-family: "katwijk_monoblack", monospace;
  display: flex;
  flex-direction: column;
  height: 100vh;
  width: 100vw;
  overflow: hidden;
}
$h: 45vh;
$side: 5vmax;
#deadline,
#lifeline {
  overflow: hidden;
  padding-inline-start: $side;
  color: black;
  flex: 1 0 $h;
  display: flex;
  flex-wrap: wrap;
  position: relative;
  .group {
    flex: 1 0 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    max-height: $h / 2;
  }
  .header {
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    position: absolute;
  }
  .value {
    text-align: end;
    //font-size: 14vh;
    font-size: min(20vh, 20vw);
    font-variant-numeric: tabular-nums;
    margin: -3rem 0;
  }
  .label {
    text-align: start;
    font-size: 5vmax;
    text-transform: uppercase;
    position: relative;
    top: 3vmax;
  }
  .side {
    font-size: 4vh;
    position: absolute;
    //left: 1vmax;
    left: 0;
    padding-left: 1vmax;
    top: 0;
    bottom: 0;
    background: black;
    margin-inline-start: -1vmax;
    width: $side;
    display: flex;
    flex-direction: column-reverse;
    justify-content: center;
    div {
      text-align: center;
      transform: rotate(-90deg);
    }
  }
}
@mixin flavor($color) {
  background: $color;
  .side {
    color: $color;
  }
}
#lifeline {
  @include flavor($lifeline);
}
#deadline {
  @include flavor($deadline);
}
#branding {
  display: flex;
  flex: 1 0 100vh - 2 * $h;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  img {
    margin: 0 1vmax;
    height: 60%;
  }
  div {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    h1,
    h4 {
      margin: 0;
    }
  }
}
</style>
