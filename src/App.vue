<template>
  <div v-if="ready" id="deadline">
    <div class="module">
      <div class="header">
        <div class="tab">DEADLINE</div>
        {{ carbon.labels[0] }}
      </div>
      <div class="groups">
        <div class="group">
          <div class="value">{{ deadlineValue.years }}</div>
          <div class="label">Year{{ deadlineValue.years > 1 ? "s" : " " }}</div>
        </div>
        <div class="group">
          <div class="value">{{ deadlineValue.days }}</div>
          <div class="label">Day{{ deadlineValue.days > 1 ? "s" : " " }}</div>
        </div>
        <div class="group">
          <div class="value">{{ deadlineValue.hours }}</div>
          <div class="label">:</div>
          <div class="value">{{ deadlineValue.minutes }}</div>
          <div class="label">:</div>
          <div class="value">{{ deadlineValue.seconds }}</div>
        </div>
      </div>
    </div>
  </div>
  <div v-if="ready" id="lifelines">
    <div class="module">
      <div class="header">
        <div class="tab">LIFELINE</div>
        {{ renewables.labels[0] }}
      </div>
      <div class="groups">
        <div class="group">
          <div class="value">{{ renewableValue }}</div>
        </div>
        <div class="group">
          <div class="label">{{ renewables.unit_labels[0] }}</div>
        </div>
      </div>
    </div>
    <div class="module">
      <div class="header">
        <div class="tab">LIFELINE</div>
        {{ gcf.labels[0] }}
      </div>
      <div class="groups">
        <div class="group">
          <div class="value">${{ gcfValue }}</div>
        </div>
        <div class="group">
          <div class="label">{{ gcf.unit_labels[0].replace("$", "") }}</div>
        </div>
      </div>
    </div>
    <div class="module">
      <div class="header">
        <div class="tab">LIFELINE</div>
        {{ gcf.labels[0] }}
      </div>
      <div class="groups">
        <div class="group">
          <div class="value">{{ indieValue }}</div>
        </div>
        <div class="group">
          <div class="label">{{ indie.unit_labels[0] }}</div>
        </div>
      </div>
    </div>
  </div>
  <div v-if="ready" id="newsfeed">
    <div class="news" one :style="animationDuration">{{ feedText }}</div>
    <div class="news" two :style="animationDuration">{{ feedText }}</div>
  </div>
  <div id="branding" @click="handleClick">
    <img src="./climateclock.svg" />
  </div>
</template>

<script>
import { DateTime } from "luxon"

const TICK_INTERVAL = 100

export default {
  data: () => ({
    ready: false,
    now: null,
    deadline: null,

    feed: "",

    newsfeed: {},
    carbon: {},
    renewables: {},
    indie: {},
    gcf: {},
    open: window.open,
  }),
  computed: {
    deadlineValue() {
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
    renewableValue() {
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
        this.gcf = modules.green_climate_fund_1
        this.indie = modules.indigenous_land_1
        this.newsfeed = modules.newsfeed_1

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

// Syntax helper for clamping a viewport unit to a min & max value
@function fluid($smallest_size, $largest_size, $viewport_unit) {
  @return max($smallest_size, min($largest_size, $viewport_unit));
}

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

@mixin flavor($color) {
  .groups {
    background: $color;
  }
  .header {
    color: $color;
    .tab {
      background-color: $color;
    }
  }
}

#deadline,
#lifelines {
  flex: 1 1 auto;
  display: flex;
  flex-direction: column;
  color: black;
}
#deadline {
  .module {
    @include flavor($deadline);
    height: 100%;
  }
  .value {
    font-size: fluid(3rem, 12rem, 15vw);
    margin: -3rem 0;
  }
  .label {
    font-size: fluid(2rem, 10rem, 6vw);
    top: 3.75vh;
  }
  .group {
    flex: 1 0 auto;
  }
}
#lifelines {
  flex: 1 1 20vh;
  .module {
    @include flavor($lifeline);
    flex: 4 0 auto;
  }
  .value {
    font-size: fluid(3rem, 10rem, 5vw);
    align-items: flex-end;
  }
  .group {
  }
  .label {
    font-size: 2vmax;
    align-items: flex-start;
  }
}
.module {
  display: flex;
  flex-direction: column;
  .header {
    font-size: fluid(1rem, 2rem, 3vw);
    display: flex;
    align-items: center;
    flex: 0 0 4vh;
    color: white;
    background: black;
    .tab {
      display: flex;
      align-items: center;
      height: 100%;
      color: black;
      padding: 0 2%;
      margin-inline-end: 2%;
    }
  }
  .groups {
    flex: 1 0 auto;
    display: flex;
    flex-wrap: wrap;
    //height: 100%;
    .group {
      display: flex;
      align-items: center;
      justify-content: center;
      max-height: $h / 2;
    }
    .value {
      text-align: end;
      font-variant-numeric: tabular-nums;
    }
    .label {
      text-align: start;
      text-transform: uppercase;
      overflow-wrap: break-word;
      position: relative;
    }
  }
}
#newsfeed {
  position: relative;
  min-height: 3rem;
  .news {
    color: $lifeline;
    font-size: fluid(1rem, 2rem, 10vw);
    text-transform: uppercase;
    white-space: nowrap;
    position: absolute;
    animation-timing-function: linear;
    animation-iteration-count: infinite;
    &[one] {
      animation-name: widget-feed-one;
    }
    &[two] {
      animation-name: widget-feed-two;
    }
  }
}
@keyframes widget-feed-one {
  0% {
    transform: translate(0, 0);
  }
  100% {
    transform: translate(-100%, 0);
  }
}
@keyframes widget-feed-two {
  0% {
    transform: translate(100%, 0);
  }
  100% {
    transform: translate(0%, 0);
  }
}
#branding {
  display: none;
  background: black;
  position: absolute;
  right: 0;
  bottom: 0;
  height: 10vh;
  /*
  display: flex;
  flex: 0 0 10vh;
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
   */
}
</style>
