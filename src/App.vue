<template>
  <div id="app">
    <calendar v-model="date"></calendar>
  </div>
</template>

<script>
import Calendar from "./components/Calendar";

export default {
    name: 'home',
    components: {
        Calendar
    },
    data() {
        return {
            date: {}
        }
    },
    watch: {
        date(newValue, oldValue) {
            console.log(newValue);
        }
    },
}
</script>
<style lang="scss">

</style>
