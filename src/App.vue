<script lang="ts">
import {defineComponent, ref} from "vue";

export default defineComponent({
  data() {
    return {
      calendar: {},
      events: Array<{
        id: string;
        title: string;
        start: Date;
      }>(),
      newEvent: {
        id: "",
        title: "",
        start: "",
      },
      dateFormat: new Intl.NumberFormat("en-US", {
        currency: "USD",
        style: "currency",
        minimumFractionDigits: 0,
      }),
      warningOpacity: 0,
      showEditEventModal: false,
    };
  },
  methods: {
    saveData(): void {
      if (this.events.length > 0) {
        const saveData = JSON.stringify(this.events);
        localStorage.setItem("saveData", saveData);
      } else {
        localStorage.removeItem("saveData");
      }
    },
    loadData(): void {
      const saveData: string | null = localStorage.getItem("saveData");
      if (saveData) {
        this.events = JSON.parse(saveData);
      }
    },
    loadCalendar(): void {
      this.calendar = new FullCalendar.Calendar(this.$refs.calendarDOM, {
        initialView: "dayGridMonth",
        headerToolbar: {
          end: "addEventButton today prev,next",
        },
        customButtons: {
          addEventButton: {
            text: "建立活動",
            click: this.EditEventModalToggle,
          },
        },
        events: this.events,
        eventClick: this.delEvent,
      });
      this.calendar.render();
    },
    EditEventModalToggle(): void {
      this.showEditEventModal = !this.showEditEventModal;
    },
    setDateTimePicker(): void {
      jQuery.datetimepicker.setLocale("zh-TW");
      jQuery(this.$refs.dateTimePickerDom).datetimepicker({
        timepicker: true,
        format: "Y-m-d H:i",
        minDate: 0,
      });
    },
    confirm(): void {
      if (this.$refs.dateTimePickerDom.value && this.newEvent.title) {
        this.newEvent.id = crypto.randomUUID();
        this.newEvent.start = this.$refs.dateTimePickerDom.value;
        this.events.push({...this.newEvent});
        this.calendar.addEvent(this.newEvent);
        this.newEvent.id = "";
        this.newEvent.start = "";
        this.newEvent.title = "";
        this.$refs.dateTimePickerDom.value = "";
        this.warningOpacity = 0;
        this.EditEventModalToggle();
      } else {
        this.warningOpacity = 1;
      }
    },
    delEvent(e): void {
      let yes = confirm("確定要刪除此筆活動嗎？");
      if (yes) {
        const index = this.events.findIndex((event) => event.id === e.event.id);
        this.events.splice(index, 1);
        e.event.remove();
      }
    },
  },
  watch: {
    events: {
      deep: true,
      handler(): void {
        this.saveData();
      },
    },
  },
  computed: {},
  mounted() {
    this.loadData();
    this.loadCalendar();
    this.setDateTimePicker();
  },
});
</script>

<template>
  <div class="body">
    <div class="container">
      <div id="calendar" ref="calendarDOM"></div>
    </div>
    <div class="Eventcontainer" v-show="showEditEventModal">
      <div class="EventInputPanel">
        <label for="inputTitle"> <h3>活動標題</h3> </label>
        <input
          id="inputTitle"
          type="text"
          placeholder="請輸入活動標題"
          v-model="newEvent.title"
        />
        <label for="inputTime"><h3>活動時間</h3></label>
        <input
          id="inputTime"
          type="text"
          ref="dateTimePickerDom"
          placeholder="請選擇活動時間"
        />
        <div>
          <p :style="{opacity: warningOpacity}">!!活動標題及時間不得為空!!</p>
          <button class="confirm-btn" @click="confirm">確定</button>
        </div>
        <button class="close-btn" @click="EditEventModalToggle">X</button>
      </div>
    </div>
  </div>
</template>

<style>
#calendar {
  width: 500px;
  aspect-ratio: 16/17;
  overflow: hidden;
  padding: 1rem;
  box-shadow: 0 0 0.2rem 0.2rem rgba(0, 242, 255, 0.75);
  /* border: 1px solid black; */
  border-radius: 1rem;
  inset: 0;
  margin: auto auto;
  position: absolute;
  background-color: white;
}

#calendar-app {
}

.Eventcontainer {
  /* display: none; */
  width: 100%;
  height: 100%;
  inset: 0;
  position: absolute;
  background-color: rgba(3, 96, 101, 0.75);
}
.EventInputPanel {
  /* font-size: small; */
  border-radius: 1rem;
  width: 200px;
  aspect-ratio: 16/9;
  inset: 0;
  margin: auto auto;
  position: absolute;
  padding: 1rem;
  border: 1px solid black;
  background-color: #fff;
  display: flex;
  gap: 1rem;
  flex-direction: column;
  justify-items: center;
}

.EventInputPanel > * {
  flex-grow: 1;
}
.EventInputPanel > label {
  display: grid;
  align-items: center;
}
.EventInputPanel > input {
  border: none;
  border-bottom: 0.1rem solid black;
}
.EventInputPanel > input:focus {
  outline: none;
}
.inputPanelTitle::after {
  content: "X";
  position: absolute;
  right: 1rem;
  top: 0;
}
.EventInputPanel p {
  font-size: small;
  margin: 0.5rem 0;
  color: rgb(249, 122, 144);
  transition: opacity 100ms linear;
}
.confirm-btn {
  width: 100%;
  height: 2rem;
  border: none;
  border-radius: 1rem;
  cursor: pointer;
  /* margin: 1rem 0rem; */
}
.close-btn {
  position: absolute;
  right: 1rem;
  cursor: pointer;
  padding: 0.3rem;
  border: none;
  border-radius: 1px;
}
.EventInputPanel button:hover {
  filter: brightness(0.9);
}
</style>
