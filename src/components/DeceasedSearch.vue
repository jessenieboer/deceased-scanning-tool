<template>

    <div>

      <div v-if="data" style="float: right; font-size: 85%"><em>When ready, drag another file here or <a href="" @click.prevent="data=null">reset all data</a></em></div>
      
      <h2>Deceased Person Search Tool</h2>

      <script type="application/javascript" src="https://cdn.jsdelivr.net/npm/xlsx@0.17.3/dist/xlsx.full.min.js"></script>

      <div v-if="loading">
          Loading spreadsheet... Please wait...
      </div>

      <div v-else-if="!data">

        <p>This tool can speed up the process of searching for people who may have been deceased prior to the Nov 2020 election. By loading in a spreadsheet of registered voters, the tool will generate the appropriate search links for FindAGrave.com and Ancestry.com (which requires a paid account) so that you can search for each person with a single click, saving time and effort.</p>

        <div class="border">
          <p>Drag and drop an XLS or CSV file onto this window to start. </p>
        </div>

        <p><a @click.prevent="showFormatHelp=!showFormatHelp" href="#">What format should the file be in?</a></p>

        <div v-if="showFormatHelp">

          <p>The current version of the tool requires that the XLS or CSV file have the following column headings in row 2 of the data (not row 1) to work correctly.</p>

          <p>Since some spreadsheets we've tested use a different set of column headings, there are some alternative labels that can also be used, shown below.</p>

          <p>Columns do NOT have to be in this order, and other columns can also be present - they will be ignored.</p>

          <table class="small">
            <tbody>
              <tr>
                <td width="10%">Column heading:</td>
                <th width="10%">First&nbsp;Name</th>
                <th width="10%" class="optional">Middle&nbsp;Name</th>
                <th width="10%">Last&nbsp;Name</th>
                <th width="10%">Birthdate</th>
                <th width="10%" class="optional">Voter ID</th>
                <th width="15%" class="optional">Voted</th>
                <th width="10%" class="optional">Address</th>
                <th width="10%" class="optional">Precinct</th>
              </tr>
              <tr>
                <td>Alternative heading:</td>
                <th>Name</th>
                <th class="optional">Name_1</th>
                <th>Name_2</th>
                <th>Birthday</th>
                <th class="optional">Unique Voter ID's</th>
                <th class="optional">Voted?</th>
                <th class="optional"></th>
                <th class="optional"></th>
              </tr>
              <tr>
                <td>Required?</td>
                <td>Required</td>
                <td class="optional">Optional</td>
                <td>Required</td>
                <td>Required</td>
                <td class="optional">Optional</td>
                <td class="optional">Optional</td>
                <td class="optional">Optional</td>
                <td class="optional">Optional</td>
              </tr>
              <tr>
                <td>Format:</td>
                <td></td>
                <td></td>
                <td></td>
                <td>mm/dd/yyyy</td>
                <td class="optional">
                  Anything
                </td>
                <td class="optional">
                  Yes / No / Blank
                </td>
                <td></td>
                <td></td>
              </tr>
              <tr>
                <td>Notes:</td>
                <td></td>
                <td></td>
                <td></td>
                <td></td>
                <td class="optional">
                  Showing this column is helpful in identifying specific voters.
                </td>
                <td class="optional">
                  The word "Yes" should indicate whether the person voted in the election. The tool provides a checkbox to display only those who voted, ignoring those who didn't. 
                  <br><br>
                  If there are multiple "<strong>Voted</strong>" or "<strong>Voted?</strong>" columns, a "Yes" in any of the first 4 will constitute a "Yes".
                </td>
                <td></td>
                <td></td>
              </tr>
            </tbody>
          </table>
        </div>

      </div>

      <div v-else>

          <h3>{{ filename }}</h3>

          <p>Found {{ data.length }} records in spreadsheet.</p>

          <div class="border flex">

            <div>
              <label class="checkbox"><input type="checkbox" v-model="filters.showVotersOnly"> Show only people who voted</label>
            </div>

            <div>
              Show people older than
              <select v-model="filters.ageLimit">
                  <option v-for="age in ageRange" :key="age" :value="age">{{ age }}</option>
              </select>
            </div>

						<!-- <div> -->
            <!--   State -->
            <!--   <select v-model="filters.state"> -->
            <!--       <option v-for="state in states" :key="state" :value="state">{{ state }}</option> -->
            <!--   </select> -->
            <!-- </div> -->

          </div>

          <p>
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
              <th>Voted</th>
              <th>Precinct</th>
              <th>Search</th>
              </tr>
          </thead>
          <tbody>
              <tr v-for="row in filteredList.slice(0, filters.maximumRecords)" :key="row['Voter ID']">
              <td>{{ row['Voter ID'] }}</td>
              <td>{{ row['First Name'] }}</td>
              <td>{{ row['Middle Name'] }}</td>
              <td>{{ row['Last Name'] }}</td>
              <td>{{ row.age }}</td>
              <td>{{ row['Birthdate'] }}</td>
              <td>
                  {{ row['Address'] || '' }}
                  {{ row['__EMPTY_1'] }}
                  {{ row['__EMPTY_2'] }}
              </td>
              <td>{{ didTheyVote(row) ? 'Yes' : '' }}</td>
              <td>{{ row['Precinct'] }}</td>
              <td>
                  <a :href="findAGraveUrl(row)" target="fag">FindAGrave</a>&nbsp;
                  <a :href="ancestryUrl(row)" target="ancestry">Ancestry</a>&nbsp;
                  <a href="#" @click.prevent="openBothWindows(row)">Both</a>
              </td>
              </tr>
          </tbody>
          </table>

          Max of {{ filters.maximumRecords }} records shown here. <a href="#" @click.prevent="filters.maximumRecords += 1000">Show more</a>.
      </div>

      <p class="muted">Tool created by [@SiWiFi](https://t.me/SiWiFi), adapted by Jesse Nieboer. <a href="https://github.com/jessenieboer/deceased-scanning-tool/" target="_blank">Code available on Github</a></p>

    </div>

</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',

  // Vue variables go here
  data: () => ({
    VERSION: '1.3.2',
    showFormatHelp: false, // whether the help section is shown
    loading: false,
    filename: null,   // filename of the dropped file
    data: null,       // the full list of records
    filters: {
      showVotersOnly: false,
ageLimit: 65,
    state: "iowa-usa", 
      maximumRecords: 1000,
    },
    votedList: null,  // the filtered list of records from 'data' that actually voted
  }),
  components: {
    // HelloWorld
  },
  watch: {
    /**
     * This function runs after we've loaded a new spreadsheet into this.data
     * We use it to filter the list to only rows that we're interested in
     */
    /* data() {
      console.time('Creating votedList');

      if (!this.data) return;

      this.votedList = this.data
        

      console.timeEnd('Creating votedList');

    }, */
  },
  computed: {
    /**
     * Filter the list by those over a specific age
     * and then sort by age (descending order)
     */
    filteredList() {
      console.time('Creating filteredList');

      if (!this.data) return;

      var result = this.data;

      if (this.filters.showVotersOnly)
        result = result.filter(this.didTheyVote)

      result = result
        .filter(item => 
            item.age > this.filters.ageLimit
        )
        .sort((a,b) => b.age - a.age);

      console.timeEnd('Creating filteredList');

      return result;
    },
    /**
     * Generate a list of ages from 0 to 120
     */
    ageRange() {
      return Array.from(Array(120).keys());
    },
   /**
     * List of states 
     */
    states() {
return Array(
"alabama-usa",
"alaska-usa",
"arizona-usa",
"arkansas-usa",
"california-usa",
"colorado-usa",
"connecticuit-usa",
"delaware-usa",
"florida-usa",
"georgia-usa",
"hawaii-usa",
"idaho-usa",
"illinois-usa",
"indiana-usa",
"iowa-usa",
"kansas-usa",
"kentucky-usa",
"louisiana-usa",
"maine-usa",
"maryland-usa",
"massachusetts-usa",
"michigan-usa",
"minnesota-usa",
"mississippi-usa",
"missouri-usa",
"montana-usa",
"nebraska-usa",
"nevada-usa",
"new+hampshire-usa",
"new+jersey-usa",
"new+mexico-usa",
"new+york-usa",
"north+carolina-usa",
"north+dakota-usa",
"ohio-usa",
"oklahoma-usa",
"oregon-usa",
"pennsylvania-usa",
"rhode+island-usa",
"south+carolina-usa",
"south+dakota-usa",
"tennesee-usa",
"texas-usa",
"utah-usa",
"vermont-usa",
"virginia-usa",
"washington-usa",
"west+virginia-usa",
"wisconsin-usa",
"wyonming-usa"
);
    },

  },
  methods: {
    /**
     * Standardize/normalize records from known formats
     */
    standardizeColumns(record) {

      // Standardize rows
      if (!record['Voter ID'])
        record['Voter ID'] = record['Unique Voter ID\'s'];

      if (!record['First Name'])
        record['First Name'] = record['Name'] || '';

      if (!record['Middle Name'])
        record['Middle Name'] = record['Name_1'] || '';

      if (!record['Last Name'])
        record['Last Name'] = record['Name_2'] || '';

      if (!record.Birthdate)
        record.Birthdate = record.Birthday;

      // Convert Excel serial dates to mm/dd/yyyy
      if (typeof record.Birthdate == 'number') {
        let bd = new Date(Date.UTC(0, 0, record.Birthdate - 1)).toISOString().split('T')[0].split('-');
        record.Birthdate = bd[1] + '/' + bd[2] + '/' + bd[0];
      }

      // Calculate age for each person
      record.age = this.age(record)
      return record;

    },
    didTheyVote(record) {
      return record['Voted'] == 'Yes' 
          || record['Voted?'] == 'Yes' 
          || record['Voted_1'] == 'Yes' 
          || record['Voted?_1'] == 'Yes' 
          || record['Voted_2'] == 'Yes'
          || record['Voted?_2'] == 'Yes'
          || record['Voted_3'] == 'Yes'
          || record['Voted?_3'] == 'Yes';
    },
    /**
     * Calculate a person's age
     */
    age(record) {
      if (!record['Birthdate'])
        return;
      if (typeof record['Birthdate'] == 'string')
        return 2021 - record['Birthdate'].split('/')[2];
      else
        console.log(record['Voter ID'], 'had an unknown birthdate type', typeof record['Birthdate'], record['Birthdate']);
    },
    /**
     * Open 2 windows at a time (some browsers will block this)
     */
    openBothWindows(item) {
      window.open(this.ancestryUrl(item), 'ancestry');
      window.open(this.findAGraveUrl(item), 'fag');
    },
    ancestryUrl(item) {
      return 'https://www.ancestry.com/search/categories/bmd_death/?name=' +
      encodeURI(item['First Name']) +
      '_' +
      encodeURI(item['Last Name']) +
      '&event=' +
      // 'columbia-boone-'
//this.filters.state +
//
      '&birth=' + this.birthYear(item) +
      '&birth_x=0-0-0&event_x=_1-0&name_x=1_1';
    },
    findAGraveUrl(item) {
      // item;
      return 'https://www.findagrave.com/memorial/search?firstname=' +
      encodeURI(item['First Name']) +
      '&lastname=' +
      encodeURI(item['Last Name']) +
      '&birthyear=' + this.birthYear(item) +
      '&birthyearfilter=' +
      // Must have died BEFORE 2021 (removed filter on request)
      //'&deathyear=2021' + 
      //'&deathyearfilter=before' + 
      '&location=&locationId=&memorialid=&mcid=&linkedToName=&datefilter=&orderby=r&plot=';
    },
    birthYear(record) {
      var year = (record.Birthdate || '').split('/')[2];
      if (year > 1905)
        return year;
      else
        return '';
    }
  },
  /**
   * On pageload, setup the drag-and-drop file read handler
   * and what to do when the file is dropped
   */
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
    holder.ondrop = async function (e) {
      
      // console.log('file dropped');
      // console.log(e);
      e.preventDefault();

      // $this.data = null;
      $this.loading = true;
      $this.filename = e.dataTransfer.files[0].name;

      console.time('nextTick triggered');

      $this.$nextTick(async function() {

        console.timeEnd('nextTick triggered');
        console.time('arrayBuffer() complete');

        const f = e.dataTransfer.files[0];
        const data = await f.arrayBuffer();

        console.timeEnd('arrayBuffer() complete');
        console.time('XLSX.read() complete');

        const workbook = window.XLSX.read(data);

        console.timeEnd('XLSX.read() complete');
        console.time('sheet_to_json() complete');

        var ssheet = workbook.Sheets[workbook.SheetNames[0]];
        $this.data = window.XLSX.utils.sheet_to_json(ssheet, {range:1});
        console.timeEnd('sheet_to_json() complete');

        console.time('standardizeColumns() complete');
        $this.data = $this.data.map(this.standardizeColumns);
        console.timeEnd('standardizeColumns() complete');


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
    };
  }
}
</script>

<style lang="scss">
  th.optional {
    background-color: #f6f6f6;
  }
  .optional {
    color: #929292;
  }
  td {
    vertical-align: top;
  }
</style>
