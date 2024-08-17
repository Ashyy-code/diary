<template>
  <!-- <header>
    <i class="las la-wine-bottle"></i>
    <span> Feed Time / Med Time</span>
  </header> -->

  <div class="loadedUser" v-if="userToken && userData">
    <nav>
      <button
        :selected="currentView == 'feed'"
        @click="
          currentView = 'feed';
          (addFeed = false), (addMeds = false);
        "
      >
        <i class="las la-utensils"></i>
      </button>
      <button
        :selected="currentView == 'settings'"
        @click="
          currentView = 'settings';
          addFeed = false;
        "
      >
        <i class="las la-cog"></i>
      </button>
      <button
        :selected="currentView == 'stats'"
        @click="
          currentView = 'stats';
          addFeed = false;
          loadStats();
        "
      >
        <i class="las la-chart-bar"></i>
      </button>
    </nav>

    <div class="cur-view">
      <span v-if="currentView == 'feed'"
        ><i class="las la-wine-bottle"></i>{{ userData.babyName }}'s Feeding
        Records</span
      >
    </div>

    <div class="feed" v-if="currentView == 'feed'">
      <div class="add">
        <div class="next">Next in: {{ feedNext }}</div>

        <button
          @click="
            addFeed = true;
            checkPreset();
          "
        >
          <i class="las la-plus-circle"></i>
        </button>
      </div>
    </div>

    <div
      class="settings animate__animated animate__fadeIn"
      v-if="currentView == 'settings'"
    >
      <h3>Settings</h3>
      <div class="setting">
        <span><h4>Baby Name</h4></span>
        <span
          ><input
            type="text"
            name
            v-model="userData.babyName"
            @change="updateSettings()"
        /></span>
      </div>
      <div class="setting">
        <span><h4>Feed Check Interval</h4></span>
        <span
          ><input
            type="number"
            v-model="userData.checkInterval"
            @change="updateSettings()"
        /></span>
      </div>
      <div class="setting">
        <span><h4>Pre-populate New Feed with Current Time</h4></span>
        <span
          ><input
            type="checkbox"
            v-model="userData.prefilTime"
            @change="updateSettings()"
        /></span>
      </div>
      <!-- <div class="setting">
        <span><h4>Enable Feed Time Notifications</h4></span>
        <span>
          <button @click="enableNotifs()">
            <i class="las la-bell"></i>
          </button>
        </span>
      </div> -->
      <div class="settingr">
        <span>Your RubyNotes Code</span><br />
        <b
          ><span>{{ userData.token }}</span></b
        >
      </div>
    </div>

    <div
      class="settings animate__animated animate__fadeIn"
      v-if="currentView == 'settings'"
    >
      <h3>Sync Data</h3>
      <div class="setting">
        <span>Enter Code</span>
        <span><input type="text" name v-model="userTokenNew" /></span>
        <button @click="forceSync()">Sync</button>
      </div>
    </div>

    <div
      class="newFeed animate__animated animate__slideInLeft animate__faster"
      v-if="addFeed"
    >
      <h3>
        <i class="las la-plus-circle"></i>
        <span>New Feed Record</span>
      </h3>
      <div class="field">
        <label for="">Time</label>
        <input
          maxlength="2"
          pattern="\d*"
          placeholder="00"
          type="tel"
          v-model="addFeedTime1"
          @focus="$event.target.select()"
          @keyup="checkmoveFeed($event.target)"
        />
        <input
          maxlength="2"
          pattern="\d*"
          placeholder="00"
          type="tel"
          v-model="addFeedTime2"
          @focus="$event.target.select()"
          ref="feedNextInp"
        />
      </div>
      <div class="field">
        <label for="">Ml</label>
        <input
          type="number"
          v-model="addFeedMl"
          @focus="$event.target.select()"
        />
      </div>
      <div class="field">
        <button @click="addFeedRecord()">Add</button>
      </div>
    </div>

    <div
      class="statbox animate__animated animate__fadeIn"
      v-if="currentView == 'stats'"
    >
      <h3>Average Stats</h3>
      <span
        >Average Feed Gap: <b> {{ avgFeedGap }}</b>
      </span>
      <span
        >Average ML per Feed: <b> {{ Math.floor(avgML) }}</b>
      </span>
      <sub>
        Average feed stats are based off the last 20 feeds only to ensure they
        are accurate for most recent baby age/situation.
      </sub>
    </div>

    <div
      v-if="currentView == 'feed'"
      class="list animate__animated animate__fadeIn"
    >
      <table>
        <thead>
          <tr>
            <th>Date</th>
            <th>Time</th>
            <th>ML</th>
            <th>Since Previous</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in userData.feedData" :key="item">
            <td>{{ item.day }}/{{ item.month }}/{{ item.year }}</td>
            <td>{{ item.time }}</td>
            <td>{{ item.ml }}</td>
            <td>{{ item.feedGap }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  mounted() {
    // localStorage.setItem("feedData","[\r\n  {\r\n    \"year\": \"2024\",\r\n    \"month\": \"8\",\r\n    \"day\": \"7\",\r\n    \"time\": \"18:45\",\r\n    \"feedGap\": \"00:00:00\",\r\n    \"ml\": 40\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 7,\r\n    \"time\": \"20:45\",\r\n    \"feedGap\": \"02:00:00\",\r\n    \"ml\": 20\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 8,\r\n    \"time\": \"02:20\",\r\n    \"feedGap\": \"05:00:00\",\r\n    \"ml\": 30\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 8,\r\n    \"time\": \"05:22\",\r\n    \"feedGap\": \"03:02:00\",\r\n    \"ml\": 50\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 8,\r\n    \"time\": \"07:50\",\r\n    \"feedGap\": \"02:30:00\",\r\n    \"ml\": 50\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 8,\r\n    \"time\": \"10:15\",\r\n    \"feedGap\": \"03:30:00\",\r\n    \"ml\": 30\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 8,\r\n    \"time\": \"12:02\",\r\n    \"feedGap\": \"01:45:00\",\r\n    \"ml\": 45\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 8,\r\n    \"time\": \"14:35\",\r\n    \"feedGap\": \"02:33:00\",\r\n    \"ml\": 20\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 8,\r\n    \"time\": \"17:40\",\r\n    \"feedGap\": \"03:05:00\",\r\n    \"ml\": 50\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 8,\r\n    \"time\": \"20:55\",\r\n    \"feedGap\": \"03:15:00\",\r\n    \"ml\": 60\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 8,\r\n    \"time\": \"22:15\",\r\n    \"feedGap\": \"02:35:00\",\r\n    \"ml\": 40\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 9,\r\n    \"time\": \"02:30\",\r\n    \"feedGap\": \"04:15:00\",\r\n    \"ml\": 60\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 9,\r\n    \"time\": \"03:45\",\r\n    \"feedGap\": \"01:15:00\",\r\n    \"ml\": 20\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 9,\r\n    \"time\": \"06:02\",\r\n    \"feedGap\": \"03:40:00\",\r\n    \"ml\": 60\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 9,\r\n    \"time\": \"08:25\",\r\n    \"feedGap\": \"02:20:00\",\r\n    \"ml\": 90\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 9,\r\n    \"time\": \"11:50\",\r\n    \"feedGap\": \"03:25:00\",\r\n    \"ml\": 45\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 9,\r\n    \"time\": \"15:40\",\r\n    \"feedGap\": \"03:50:00\",\r\n    \"ml\": 45\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 9,\r\n    \"time\": \"18:15\",\r\n    \"feedGap\": \"03:30:00\",\r\n    \"ml\": 85\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 9,\r\n    \"time\": \"21:10\",\r\n    \"feedGap\": \"03:05:00\",\r\n    \"ml\": 65\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 10,\r\n    \"time\": \"01:00\",\r\n    \"feedGap\": \"04:00:00\",\r\n    \"ml\": 75\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 10,\r\n    \"time\": \"04:05\",\r\n    \"feedGap\": \"03:00:00\",\r\n    \"ml\": 50\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 10,\r\n    \"time\": \"07:20\",\r\n    \"feedGap\": \"03:20:00\",\r\n    \"ml\": 45\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 10,\r\n    \"time\": \"10:05\",\r\n    \"feedGap\": \"0:20:00\",\r\n    \"ml\": 80\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 10,\r\n    \"time\": \"11:35\",\r\n    \"feedGap\": \"0:15:00\",\r\n    \"ml\": 30\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 10,\r\n    \"time\": \"14:20\",\r\n    \"feedGap\": \"03:00:00\",\r\n    \"ml\": 60\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 10,\r\n    \"time\": \"15:45\",\r\n    \"feedGap\": \"01:20:00\",\r\n    \"ml\": 75\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 10,\r\n    \"time\": \"17:50\",\r\n    \"feedGap\": \"02:10:00\",\r\n    \"ml\": 60\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 10,\r\n    \"time\": \"20:05\",\r\n    \"feedGap\": \"02:15:00\",\r\n    \"ml\": 50\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 10,\r\n    \"time\": \"21:47\",\r\n    \"feedGap\": \"01:20:00\",\r\n    \"ml\": 40\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 10,\r\n    \"time\": \"22:20\",\r\n    \"feedGap\": \"01:20:00\",\r\n    \"ml\": 30\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 10,\r\n    \"time\": \"23:08\",\r\n    \"feedGap\": \"00:48:00\",\r\n    \"ml\": 20\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 11,\r\n    \"time\": \"04:17\",\r\n    \"feedGap\": \"04:20:00\",\r\n    \"ml\": 80\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 11,\r\n    \"time\": \"08:05\",\r\n    \"feedGap\": \"03:45:00\",\r\n    \"ml\": 100\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 11,\r\n    \"time\": \"11:45\",\r\n    \"feedGap\": \"03:45:00\",\r\n    \"ml\": 100\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 11,\r\n    \"time\": \"14:30\",\r\n    \"feedGap\": \"02:50:00\",\r\n    \"ml\": 70\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 11,\r\n    \"time\": \"17:30\",\r\n    \"feedGap\": \"03:45:00\",\r\n    \"ml\": 90\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 12,\r\n    \"time\": \"20:45\",\r\n    \"feedGap\": \"03:45:00\",\r\n    \"ml\": 90\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 13,\r\n    \"time\": \"00:30\",\r\n    \"feedGap\": \"03:00:00\",\r\n    \"ml\": 100\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 13,\r\n    \"time\": \"03:30\",\r\n    \"feedGap\": \"03:00:00\",\r\n    \"ml\": 100\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 13,\r\n    \"time\": \"06:15\",\r\n    \"feedGap\": \"03:00:00\",\r\n    \"ml\": 60\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 13,\r\n    \"time\": \"09:00\",\r\n    \"feedGap\": \"02:45:00\",\r\n    \"ml\": 100\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 13,\r\n    \"time\": \"11:30\",\r\n    \"feedGap\": \"02:30:00\",\r\n    \"ml\": 90\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 13,\r\n    \"time\": \"15:30\",\r\n    \"feedGap\": \"04:00:00\",\r\n    \"ml\": 85\r\n  },\r\n  {\r\n    \"year\": 2024,\r\n    \"month\": 8,\r\n    \"day\": 13,\r\n    \"time\": \"16:00\",\r\n    \"feedGap\": \"00:30:00\",\r\n    \"ml\": 30\r\n  }\r\n]");

    //localStorage.setItem("feedData",JSON.stringify(this.feedData.reverse()));

    // let feedSettings = {
    //   "babyName":"Ruby",
    //   "intervalCheck":3,
    //   "prefilTime":false
    // }

    // localStorage.setItem("feedSettings",JSON.stringify(feedSettings));

    //console.log(localStorage.getItem("feedData"));

    //this.syncFeedData();

    //this.createFeedDataUser();

    //check for a RN token existance
    if (localStorage.getItem("rubyNotesToken")) {
      this.userToken = localStorage.getItem("rubyNotesToken");
      this.userTokenNew = localStorage.getItem("rubyNotesToken");
      if (this.userToken) {
        //we have a token, load the data
        //set the user data
        this.getFeedData();
      }
    } else {
      //begin signup phase here:
      this.createFeedDataUser();
      this.beginTimer();
    }
  },

  data() {
    return {
      //feedData: JSON.parse(localStorage.getItem("feedData")),
      //feedSettings: JSON.parse(localStorage.getItem("feedSettings")),
      userData: null,
      currentView: "feed",
      feedNext: "00:00",
      medNext: "00:00",
      addFeed: false,
      addMed: false,
      addFeedTime1: "00",
      addFeedTime2: "00",
      addFeedMl: 0,
      //stats#
      avgML: 0,
      avgFeedGap: "00:00:00",
      userToken: null,
      userTokenNew: null,
      appLoaded: false,
      createUser: false,
      createUserBabyName: null,
      createUserInterval: null,
      timer: 1,
    };
  },

  methods: {
    checkmoveFeed(ctrl) {
      if (ctrl.value.length > 1) {
        this.$refs.feedNextInp.focus();
      }
    },

    addHours(date, hours) {
      const hoursToAdd = hours * 60 * 60 * 1000;
      date.setTime(date.getTime() + hoursToAdd);
      return date;
    },

    addFeedRecord() {
      //catch for first record
      if (this.userData.feedData.length == 0) {
        let feedRecord = {
          time: this.addFeedTime1 + ":" + this.addFeedTime2,
          ml: this.addFeedMl,
          feedGap: "00:00:00",
          year: new Date().getYear() + 1900,
          month: new Date().getMonth() + 1,
          day: new Date().getDate(),
        };
        this.userData.feedData.unshift(feedRecord);
        this.syncFeedData();
        //localStorage.setItem("feedData", JSON.stringify(this.feedData));
        this.addFeed = false;
        return;
      }

      //get the latest feed record
      let recentFeed = this.createDateObject(
        this.userData.feedData[0].year,
        this.userData.feedData[0].month - 1,
        this.userData.feedData[0].day,
        this.userData.feedData[0].time,
        0
      );
      let rightFuckingNow = new Date();
      let thisTime = this.createDateObject(
        rightFuckingNow.getFullYear(),
        rightFuckingNow.getMonth(),
        rightFuckingNow.getDate(),
        this.addFeedTime1 + ":" + this.addFeedTime2,
        0
      );

      let diff = thisTime - recentFeed;
      let feedGapper = this.convertMillisecondsToTime(diff);

      console.log(thisTime);
      console.log(recentFeed);
      console.log(diff);

      let feedRecord = {
        time: this.addFeedTime1 + ":" + this.addFeedTime2,
        ml: this.addFeedMl,
        feedGap: feedGapper,
        year: new Date().getYear() + 1900,
        month: new Date().getMonth() + 1,
        day: new Date().getDate(),
      };
      this.userData.feedData.unshift(feedRecord);
      this.syncFeedData();
      //localStorage.setItem("feedData", JSON.stringify(this.feedData));
      this.addFeed = false;
    },

    createDateObject(year, month, day, time, add) {
      // Parse the time string to get hours and minutes
      const [hours, minutes] = time.split(":").map(Number);

      // Create and return the new Date object
      return new Date(year, month, day, hours + add, minutes);
    },

    convertMillisecondsToTime(ms) {
      // Convert milliseconds to total seconds
      const totalSeconds = Math.floor(ms / 1000);

      // Calculate hours, minutes, and remaining seconds
      const hours = Math.floor(totalSeconds / 3600);
      const minutes = Math.floor((totalSeconds % 3600) / 60);
      const seconds = totalSeconds % 60;

      // Format hours, minutes, and seconds to ensure they are always two digits
      const formattedHours = String(hours).padStart(2, "0");
      const formattedMinutes = String(minutes).padStart(2, "0");
      const formattedSeconds = String(seconds).padStart(2, "0");

      // Return the formatted time string
      return `${formattedHours}:${formattedMinutes}:${formattedSeconds}`;
    },

    secondsToTime(seconds) {
      const hours = Math.floor(seconds / 3600);
      seconds %= 3600;
      const minutes = Math.floor(seconds / 60);
      const sec = Math.floor(seconds % 60);
      return `${String(hours).padStart(2, "0")}:${String(minutes).padStart(
        2,
        "0"
      )}:${String(sec).padStart(2, "0")}`;
    },
    timeToSeconds(time) {
      const [hours, minutes, seconds] = time.split(":").map(Number);
      return hours * 3600 + minutes * 60 + seconds;
    },

    loadStats() {
      let mostRecentFeeds = this.userData.feedData.slice(0, 30);

      // Calculate the average "ml"
      const totalMl = mostRecentFeeds.reduce(
        (sum, session) => sum + session.ml,
        0
      );
      const averageMl = totalMl / mostRecentFeeds.length;

      // Helper function to convert HH:MM:SS to seconds

      // Calculate the average "feedGap"
      const totalFeedGapInSeconds = mostRecentFeeds.reduce(
        (sum, session) => sum + this.timeToSeconds(session.feedGap),
        0
      );
      const averageFeedGapInSeconds =
        totalFeedGapInSeconds / mostRecentFeeds.length;

      // Helper function to convert seconds back to HH:MM:SS

      const averageFeedGap = this.secondsToTime(averageFeedGapInSeconds);

      //console.log("Average ml:", averageMl);
      //console.log("Average feedGap:", averageFeedGap);

      this.avgFeedGap = averageFeedGap;
      this.avgML = averageMl;
    },
    updateSettings() {
      //localStorage.setItem("feedSettings", JSON.stringify(this.feedSettings));
      this.syncFeedData();
    },
    checkPreset() {
      if (this.userData.prefilTime) {
        this.addFeedTime1 = this.getCurrentTime().hour;
        this.addFeedTime2 = this.getCurrentTime().mins;
        // alert(this.addFeedTime1)
      } else {
        this.addFeedTime1 = "00";
        this.addFeedTime2 = "00";
      }
    },
    getCurrentTime() {
      const now = new Date();

      // Get the current hour (24-hour format)
      let hour = now.getHours();

      // Get the minutes past the hour
      let mins = now.getMinutes();

      // Format hour and minutes as two digits
      hour = hour < 10 ? "0" + hour : hour;
      mins = mins < 10 ? "0" + mins : mins;

      return { hour, mins };
    },

    async getFeedData() {
      await axios
        .post(
          "https://ashypls-001-site1.ftempurl.com/RubyNotes.asmx/getUserNotes",
          {
            contentType: "application/json",
            userToken: this.userToken,
          }
        )
        .then((response) => {
          if (response.data === "") {
            console.log("not found");
          } else {
            this.userData = response.data;
            //this.userData.feedData = response.data.feedData.reverse();
            console.log(this.userData);
            this.beginTimer();
            //this.userToken = this.userData.token;
            //localStorage.setItem("rubyNotesToken",this.userToken);
            //this.syncFeedData();
          }
          this.appLoaded = true;
          //console.log(response.data);
        });
    },

    //sync the data --------------------------------------------------------------------------------
    async syncFeedData() {
      await axios
        .post(
          "https://ashypls-001-site1.ftempurl.com/RubyNotes.asmx/syncUserNotes",
          {
            contentType: "application/json",
            userToken: this.userToken,
            feedData: JSON.stringify(this.userData),
          }
        )
        .then((response) => {
          //check
          if (this.userData.feedData.length == 1) {
            this.beginTimer();
          }
        });
    },

    //create new user
    async createFeedDataUser() {
      await axios
        .post(
          "https://ashypls-001-site1.ftempurl.com/RubyNotes.asmx/createUser",
          {
            contentType: "application/json",
          }
        )
        .then((response) => {
          console.log(response.data);
          this.userData = response.data;
          this.userToken = this.userData.token;
          console.log(this.userToken);
          //set
          localStorage.setItem("rubyNotesToken", this.userToken);
        });
    },

    beginTimer() {
      if (!this.userData) {
        console.log(this.userData);
        return;
      }
      if (this.userData.feedData.length == 0) {
        console.log(this.userData);
        return;
      }

      setInterval(() => {
        //get the most recent feed time

        let recentFeed = this.createDateObject(
          this.userData.feedData[0].year,
          this.userData.feedData[0].month - 1,
          this.userData.feedData[0].day,
          this.userData.feedData[0].time,
          this.userData.checkInterval
        );
        let rightFuckingNow = new Date();

        // console.log(this.feedData[0]);

        let diff = recentFeed - rightFuckingNow;
        if (diff.toString().startsWith("-")) {
          this.feedNext = "OVERDUE";
        } else {
          this.feedNext = this.convertMillisecondsToTime(diff);
        }

        // console.log("tick");
        //this.timer =  this.userData;
      }, 1000);
    },

    forceSync() {
      //set the new token
      localStorage.setItem("rubyNotesToken", this.userTokenNew);
      window.location = "/";
    },

    enableNotifs() {
      Notification.requestPermission().then((result) => {
        console.log(result);
      });
    },
  },
};
</script>

<style lang="scss">
body,
html {
  margin: 0;
  padding: 0;
  overflow: hidden;
  background: #333;
}
#app {
  height: 100vh;
  width: 100vw;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  overflow-y: auto;
  color: wheat;
  display: flex;
  flex-direction: column;
}
* {
  box-sizing: border-box;
}
header {
  display: flex;
  gap: 0.5rem;
  justify-content: center;
  background: wheat;
  color: #333;
  padding: 1rem;
  font-size: 1.5rem;
  align-items: center;

  i {
    font-size: 2rem;
  }
}
nav {
  display: flex;
  justify-content: space-around;
  height: 50px;
  padding: 0.5rem;
  width: calc(100vw);
  gap: 0.5rem;
  position: absolute;
  bottom: 0;
  background: rgba(0, 0, 0, 0.854);
  height: calc(50px + 1rem);
  z-index: 999999;

  button {
    height: 50px;
    width: 50px;
    border: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    background: rgb(84, 81, 81);
    color: white;
    transition: all 200ms ease;
    border-radius: 50%;

    &[selected="true"] {
      background: rgb(185, 154, 95);
    }

    i {
      font-size: 200%;
    }
  }
}
.cur-view {
  color: wheat;
  padding: 0.5rem;
  font-size: 1.3rem;
  margin-top: 1rem;
}
.feed {
  display: flex;
  flex-direction: column;

  .add {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem;
    gap: 2rem;

    .next {
      background: rgb(233, 75, 75);
      flex-grow: 1;
      font-size: 1.8rem;
      padding: 0.25rem;
      border-radius: 0.5rem;
    }

    button {
      height: 40px;
      width: 40px;
      border: 0;
      border-radius: 50%;
      position: relative;
      background: none;

      i {
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        color: rgb(102, 228, 102);
        font-size: 300%;
      }
    }
  }
}
table {
  width: calc(100%);
  border: solid 3px wheat;
  border-collapse: collapse;

  thead {
    tr {
      background: wheat;
      color: rgb(84, 69, 41);
      th {
        border-right: solid 1px #333;
      }
    }
  }
  tbody {
    td {
      padding: 0.5rem;
      border: solid 1px wheat;
    }
  }
}

.newFeed {
  background: rgb(46, 45, 43);
  padding: 1rem;
  margin: 1rem;
  margin-right: 1rem;
  margin-top: 0;
  border-radius: 0.5rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
  border: solid 3px wheat;

  h3 {
    margin: 0;
    padding: 0;
    display: flex;
    gap: 0.5rem;
    justify-content: center;
    color: rgb(92, 214, 92);
  }

  .field {
    display: flex;
    gap: 0.5rem;
    align-items: center;
    font-size: 120%;
    justify-content: space-between;

    button {
      border: 0;
      padding: 0.5rem;
      border-radius: 0.25rem;
      background: wheat;
      font-size: 1.2rem;
      width: 100%;
    }
  }
}
.list {
  margin-bottom: 175px;
}
.statbox {
  sub {
    margin-top: 1rem;
    color: white;
  }

  h3 {
    margin: 0;
    padding: 0;
    margin-bottom: 1rem;
  }

  span {
    margin-bottom: 0.5rem;
    padding: 1rem;
    border-radius: 0.5rem;
    background: rgb(22, 22, 22);

    b {
      color: rgb(254, 252, 107);
    }
  }

  border: solid 3px wheat;
  display: flex;
  flex-direction: column;
  padding: 1rem;
  margin: 1rem;
  background: rgb(35, 34, 34);
  border-radius: 0.5rem;
}

.settings {
  border: solid 3px wheat;
  display: flex;
  flex-direction: column;
  padding: 1rem;
  margin: 1rem;
  background: rgb(35, 34, 34);
  border-radius: 0.5rem;

  b {
    margin-top: 1rem;
    color: white;
  }

  .setting {
    background: rgb(22, 22, 22);
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    padding: 1rem;
    text-align: left;
    margin-bottom: 0.5rem;
    border-radius: 0.5rem;
  }

  h3,
  h4 {
    padding: 0;
    margin: 0;
  }

  h3 {
    width: 100%;
    text-align: center;
    margin-bottom: 1rem;
  }
}
input {
  background: rgb(88, 86, 86);
  border: 0;
  padding: 0.5rem;
  font-size: 120%;
  width: 100px;
  color: wheat;
  &[name] {
    width: 150px;
  }
}
</style>
