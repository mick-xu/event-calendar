# events-calendar
A event calendar component for Vue

<img src="https://raw.githubusercontent.com/mick-xu/event-calendar/master/screenshot.png" />

### Features
- Display events

### Installation
```
npm install events-calendar
# or
yarn add events-calendar
```
### Usage
```
import Calendar from 'events-calendar'
import 'events-calendar/lib/events-calendar.css'

components: {
    Calendar
},
data() {
    return {
        date: {}
    }
},
```

```
<template>
  <Calendar v-model="date" />
</template>
```
