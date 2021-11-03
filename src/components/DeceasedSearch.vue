<template>

    <div>

      <h2>Deceased Person Search Tool</h2>
      <script type="application/javascript" src="https://cdn.jsdelivr.net/npm/xlsx@0.17.3/dist/xlsx.full.min.js"></script>
      <div v-if="loading">
          Loading spreadsheet... Please wait...
      </div>
      <div v-else-if="!data" style="border: 1px solid #aaa; border-radius: 5px; padding: 18px 24px;">
          <p>Drag and drop an XLS or CSV file onto this window to start. </p>
      
          <p>It MUST be in the Missouri "dropped voters" format where there are exactly 2 header rows, and the columns are named consistently to the Barry/Boone/Buchanan sheets.</p>
      </div>
      <div v-else>
          <p>Found {{ data.length }} records in spreadsheet.</p>
          <p>
          Here are the ones who voted, and are older than
          <select v-model="ageLimit">
              <option v-for="age in ageRange" :key="age" :value="age">{{ age }}</option>
          </select>.
          Found {{ filteredList.length }} of those.
          </p>
          <ul>
          <li>Clicking the search links will initiate a search for the specific person using the provided name, birthdate and state.</li>
          <li>In order to search Ancestry.com you MUST be logged in to their site and have a paid subscription.</li>
          <li>To speed the process up, after running a search, drag the position of all 3 windows so you can see them on the screen together. Then when you click the next search link it will reuse the same window(s) as before.</li>
          <li>Links will change colour (to purple) after you've run that search, so you can keep track of which ones you've reviewed.</li>
          <li>You cannot record your findings on this page (as yet). You will need to note them down in the original spreadsheet.</li>
          </ul>
          <table>
          <thead>
              <tr>
              <th>Voter ID</th>
              <th>First Name</th>
              <th>Middle Name</th>
              <th>Last Name</th>
              <th>Age</th>
              <th>Birthdate</th>
              <th>Address</th>
              <th>Precinct</th>
              <th>Search</th>
              </tr>
          </thead>
          <tbody>
              <tr v-for="row in filteredList.slice(0,1000)" :key="row['Unique Voter ID\'s']">
              <td>{{ row['Voter ID'] || row['Unique Voter ID\'s'] }}</td>
              <td>{{ row['Name'] }}</td>
              <td>{{ row['Name_1'] }}</td>
              <td>{{ row['Name_2'] }}</td>
              <td>{{ age(row) }}</td>
              <td>{{ row['Birthdate'] }}</td>
              <td>
                  {{ row['Address'] }}
                  {{ row['__EMPTY_1'] }}
                  {{ row['__EMPTY_2'] }}
              </td>
              <td>{{ row['Precinct'] }}</td>
              <td>
                  <a :href="findAGraveUrl(row)" target="fag">FindAGrave</a>&nbsp;
                  <a :href="ancestryUrl(row)" target="ancestry">Ancestry</a>&nbsp;
                  <a href="#" @click.prevent="openBothWindows(row)">Both</a>
              </td>
              </tr>
          </tbody>
          </table>
          Max of 1000 records shown here.
      </div>
      <p><small><em>Version {{ VERSION }}. This is still very rough and there may be bugs. If this proves useful, we may expand the features and support for additional data. Send feedback via <a href="https://t.me/SiWiFi">@SiWiFi</a> on Telegram.</em></small></p>

    </div>

</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  data: () => ({
    VERSION: '1.1.0',
    loading: false,
    ssheet: null,
    data: null,
    ageLimit: 80,
    votedList: null,
  }),
  components: {
    // HelloWorld
  },
  watch: {
    data() {
      this.votedList = this.data && this.data
        .filter(item => item['Voted_1'] == 'Yes' || item['Voted?_1'] == 'Yes')
        .map(item => {
          // Standardize rows
          if (!item.Birthdate)
            item.Birthdate = item.Birthday;
          item.age = this.age(item)
          return item;
        });
    },
  },
  computed: {
    filteredList() {
      return this.votedList && this.votedList
        .filter(item => 
            item.age > this.ageLimit
        )
        .sort((a,b) => b.age - a.age);
    },
    ageRange() {
      return Array.from(Array(120).keys());
    }
  },
  methods: {
    age(record) {
      return 2021 - record['Birthdate'].split('/')[2];
    },    
    openBothWindows(item) {
      window.open(this.ancestryUrl(item), 'ancestry');
      window.open(this.findAGraveUrl(item), 'fag');
    },
    ancestryUrl(item) {
      return 'https://www.ancestry.com/search/categories/bmd_death/?name=' +
      encodeURI(item.Name + ' ' + item.Name_1) +
      '_' +
      encodeURI(item.Name_2) +
      '&event=_' +
      // 'columbia-boone-'
      'missouri-usa' +
      '&birth=' +
      (item.Birthday || item.Birthdate || '').split('/')[2] +
      '&birth_x=0-0-0&name_x=1_1';
    },
    findAGraveUrl(item) {
      item;
      return 'https://www.findagrave.com/memorial/search?firstname=' +
      encodeURI(item.Name) +
      '&middlename=' +
      encodeURI(item.Name_1 || '') +
      '&lastname=' +
      encodeURI(item.Name_2) +
      '&birthyear=' +
      (item.Birthday || item.Birthdate || '').split('/')[2] +
      '&birthyearfilter=' +
      // Must have died BEFORE 2021
      '&deathyear=2021' + 
      '&deathyearfilter=before' + 
      '&location=&locationId=&memorialid=&mcid=&linkedToName=&datefilter=&orderby=r&plot=';
    },
  },
  mounted() {
    console.log('mounted');
    var $this = this;
    window.VueApp = this;

    var holder = document.querySelector('body');
    
    holder.ondragover = function (e) { 
      e.preventDefault();
      this.className = 'hover'; return false; 
    };
    // holder.ondragend = function () { this.className = ''; return false; };
    holder.addEventListener('drop', async function (e) {
      
      // console.log('file dropped');
      // console.log(e);
      e.preventDefault();

      // $this.data = null;
      $this.loading = true;

      $this.$nextTick(async function() {

        const f = e.dataTransfer.files[0];
        const data = await f.arrayBuffer();
        const workbook = window.XLSX.read(data);
        var ssheet = workbook.Sheets[workbook.SheetNames[0]];
        $this.data = window.XLSX.utils.sheet_to_json(ssheet, {range:1});
        $this.loading = false;

        // delete f;
        // delete data;
        // delete workbook;
        // delete ssheet;

      });


      // var headerRows = 2;


      // var file = e.dataTransfer.files[0],
      //     reader = new FileReader();
      // reader.onload = function (event) {
      //   console.log(event.target);
      //   var sheet = window.XLSX.read(event.target.result);
        
      //   // holder.style.background = 'url(' + event.target.result + ') no-repeat center';
      // };
      // console.log(file);
      // reader.readAsDataURL(file);

      // return false;
    });
  }
}
</script>

<style>

</style>
