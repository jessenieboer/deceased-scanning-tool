<template>
  <div id="app">

    <deceased-search></deceased-search>

  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import DeceasedSearch from './components/DeceasedSearch.vue'

export default {
  name: 'App',
  data: () => ({
    loading: false,
    ssheet: null,
    data: null,
    ageLimit: 80,
    votedList: null,
  }),
  components: {
    // HelloWorld
    DeceasedSearch
  },
}
</script>

<style lang="scss">
html, body {
	height: 100%;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* text-align: center; */
  color: #2c3e50;
  /* margin-top: 60px; */
}
table {
  border-collapse: collapse;
  margin-bottom: 24px;
  font-size: 0.9rem;

  &.small {
    font-size: 12px;
  }

  tr:hover td {
    background-color: #f2f2f2;
  }
  td, th {
    border: 1px solid #ccc;
    padding: 2px 4px;
  }
  th {
    font-weight: bold;
    background-color: #eee;
    font-size: 12px;
  }
}
.small {
  font-size: 85%;
}
.muted {
  opacity: 0.6;
}
.border {
  border: 1px solid #aaa; 
  border-radius: 5px; 
  padding: 18px 24px; 
  margin-bottom: 16px;
}
label.checkbox {
  cursor: pointer;
}
.flex {
  display: flex;
  align-items: baseline;
}
.flex > * {
  margin-right: 24px;
}
input, select {
  /* vertical-align: middle; */
}
</style>
