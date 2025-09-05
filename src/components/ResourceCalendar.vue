<template>
  <div class="wrap">
    <div class="left">
      <DayPilotNavigator
          ref="navigator"
          :showMonths="3"
          :skipMonths="3"
          selectMode="Day"
          :startDate="initialNavigatorDate"
          :selectionDay="initialNavigatorDate"
          :onTimeRangeSelected="onNavTimeRangeSelected"
          :events="events"
      />
    </div>
    <div class="content">
      <DayPilotCalendar
          ref="calendar"
          viewType="Resources"
          :events="events"
          :columns="columns"
          :startDate="startDate"
          timeRangeSelectedHandling="Enabled"
          :onTimeRangeSelected="onCalTimeRangeSelected"
          :onEventClick="onEventClick"
          eventDeleteHandling="Disabled"
          :onEventMoved="onEventMoved"
          :onEventResized="onEventResized"
          :eventBorderRadius="4"
          :onBeforeEventRender="onBeforeEventRender"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue"
import { DayPilot, DayPilotCalendar, DayPilotNavigator } from "@daypilot/daypilot-lite-vue"

const calendar = ref(null)
const navigator = ref(null)

const initialNavigatorDate = "2026-09-01"
const startDate = ref("2026-09-01")

const events = ref([])
const columns = ref([])

const onNavTimeRangeSelected = (args) => {
  startDate.value = args.day
}

const onCalTimeRangeSelected = async (args) => {
  await createEvent(args.start, args.end, args.resource)
  calendar.value.control.clearSelection()
}

const onEventClick = (args) => {
  editEvent(args.e)
}
const onEventMoved = (args) => {
  console.log("Event moved", args.e)
}
const onEventResized = (args) => {
  console.log("Event resized", args.e)
}

const onBeforeEventRender = (args) => {
  if (!args.data.backColor) {
    args.data.backColor = "#bbbbbb"
  }
  args.data.borderColor = "darker"
  args.data.html = ""
  args.data.barHidden = true
  args.data.areas = [
    {
      left: 4,
      height: 24,
      top: 4,
      right: 4,
      html: args.data.text,
      padding: 2,
      horizontalAlignment: "center",
      backColor: "#fff",
      style: "opacity: 0.5"
    }
  ]
}

const loadEvents = () => {
  events.value = [
    { id: 1, start: "2026-09-01T10:00:00", end: "2026-09-01T11:00:00", resource: "R3", text: "Event 1", backColor: "#6aa84fcc" },
    { id: 2, start: "2026-09-01T13:00:00", end: "2026-09-01T16:00:00", resource: "R3", text: "Event 2", backColor: "#f1c232cc" },
    { id: 3, start: "2026-09-01T13:30:00", end: "2026-09-01T16:30:00", resource: "R2", text: "Event 3", backColor: "#cc4125cc" },
    { id: 4, start: "2026-09-01T10:30:00", end: "2026-09-01T12:30:00", resource: "R2", text: "Event 4" },
    { id: 5, start: "2026-09-10T10:30:00", end: "2026-09-10T12:30:00", resource: "R2", text: "Event 5" },
    { id: 6, start: "2026-09-18T10:30:00", end: "2026-09-18T12:30:00", resource: "R2", text: "Event 6" },
    { id: 7, start: "2026-09-19T10:30:00", end: "2026-09-19T12:30:00", resource: "R2", text: "Event 7" },
  ]
}

const loadResources = () => {
  columns.value = [
    { name: "Resource 1", id: "R1" },
    { name: "Resource 2", id: "R2" },
    { name: "Resource 3", id: "R3" },
    { name: "Resource 4", id: "R4" },
    { name: "Resource 5", id: "R5" },
    { name: "Resource 6", id: "R6" },
  ]
}

const editEvent = async (e) => {
  const form = [
    { name: "Text", id: "text" },
    { name: "Start", id: "start", type: "datetime", disabled: true },
    { name: "End", id: "end", type: "datetime", disabled: true },
    { name: "Resource", id: "resource", type: "select", options: calendar.value.control.columns.list },
  ]
  const data = e.data
  const modal = await DayPilot.Modal.form(form, data)
  if (modal.canceled) {
    return
  }

  calendar.value.control.events.update(modal.result)
}

const createEvent = async (start, end, resource) => {
  const form = [
    { name: "Text", id: "text" },
    { name: "Start", id: "start", type: "datetime", disabled: true },
    { name: "End", id: "end", type: "datetime", disabled: true },
    { name: "Resource", id: "resource", type: "select", options: calendar.value.control.columns.list },
  ]
  const data = { start, end, resource, id: DayPilot.guid() }
  const modal = await DayPilot.Modal.form(form, data)
  if (modal.canceled) {
    return
  }

  const e = modal.result
  calendar.value.control.events.add(e)
}

onMounted(() => {
  loadResources()
  loadEvents()
})
</script>

<style>
.wrap {
  display: flex;
}
.left {
  margin-right: 10px;
}
.content {
  flex-grow: 1;
}
.navigator_default_busy.navigator_default_cell {
  border-bottom: 4px solid #ee4f2ecc;
  box-sizing: border-box;
}
</style>
