<!-- https://medium.com/@ravipatel.it/step-by-step-guide-reading-public-google-sheets-data-using-javascript-and-displaying-it-on-an-html-f6aee8416a9c -->

<template>
  <q-page>
    <div class="row items-center justify-evenly q-mt-lg">
      <q-stepper
        v-model="stepIdx"
        vertical
        color="primary"
        animated
        class="row"
      >
        <q-step
          v-for="s in steps"
          :key="steps.indexOf(s)"
          :name="steps.indexOf(s)"
          :title=getStep(s).prompt
          icon="settings"
          :done="stepIdx > steps.indexOf(s)"
        >
          <!-- {{ getStep(s).tags }} - {{ getStep(s).dayNight }} - {{ getStep(s).includeOne }} -->
          <q-stepper-navigation>
            <q-btn @click="stepIdx += 1" color="primary" label="Continue" />
          </q-stepper-navigation>
        </q-step>
      </q-stepper>
    </div>

    <div class="row items-center justify-evenly q-my-lg">
      <q-btn color="green" label="Go!" @click="fetchGoogleSheetData" />
    </div>

    <div class="row items-center justify-evenly q-my-lg">
      <q-btn color="purple" label="Settings">
        <q-menu>
          <div class="row no-wrap q-pa-md">
            <div class="column">
              <div class="text-h6 q-mb-md">Settings</div>
              <q-select v-model="dayNight" :options="['Day', 'Night', 'Both']" label="Day/Night"  style="min-width: 200px" />
              <q-toggle v-model="includeSexy" label="Night" />
            </div>
          </div>
        </q-menu>
      </q-btn>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { onMounted, Ref, ref } from 'vue'
import { Step } from 'src/types'

const spreadsheetId = '1AbZ0UlsIBKXt6AIvyQDCqasNZTZK-TXD70drcxrsI-8'

const apiKey = import.meta.env.VITE_GOOGLE_API_KEY;

const url = `https://sheets.googleapis.com/v4/spreadsheets/${spreadsheetId}/values/Sheet1?key=${apiKey}`;

let steps = ref([]) as Ref<Array<Array<string>>>
let stepIdx = ref(0)

let dayNight = ref('Both')
let tagsToExclude = ref(['Needs Work'])
let includeOne = ref('Sexy')
let includeSexy = ref(false)

async function fetchGoogleSheetData() {
  try {
    // Fetch data from Google Sheets API
    const response = await fetch(url);
    const data = await response.json();
    
    let rows = data.values; // Extract rows from the data
    rows.shift(); // Remove the header row

    // Filter out day/night steps
    if (dayNight.value === 'Day') {
      rows = rows.filter((s: Array<string>) => getStep(s).dayNight !== 'Night')
    } else if (dayNight.value === 'Night') {
      rows = rows.filter((s: Array<string>) => getStep(s).dayNight !== 'Day')
    }

    // Filter out tags to exclude
    rows = rows.filter((s: Array<string>) => !tagsToExclude.value.includes(getStep(s).tags))

    // If there's an includeOne tag, grab it
    let stepToInclude = null
    if (includeSexy.value && includeOne.value !== '') {
      const includeOneSteps = rows.filter((s: Array<string>) => getStep(s).includeOne === includeOne.value)
      if (includeOneSteps.length > 0) {
        stepToInclude = includeOneSteps[Math.floor(Math.random() * includeOneSteps.length)]
        const step = getStep(stepToInclude)
        // Remove the step to include from the rows
        rows = rows.filter((s: Array<string>) => getStep(s).prompt !== step.prompt)
      }
    }

    // Shuffle the remaining rows
    const shuffledRows = rows.sort(() => Math.random() - 0.5)

    // Select 5 from the shuffled rows
    if (stepToInclude !== null) {
      // Put the special step to include last
      steps.value = [...shuffledRows.slice(0, 4), stepToInclude]
    } else {
      steps.value = shuffledRows.slice(0, 5)
    }

    // // Get the table body element
    // const tableBody = document.querySelector('#data-table tbody');
    // if (!tableBody) {
    //     throw new Error('Table body element not found');
    // }

    // // Loop through the rows (starting from row 1 to skip headers)
    // for (let i = 1; i < rows.length; i++) {
    //     const row = document.createElement('tr');
        
    //     // Loop through each cell in the row and create a table cell for each
    //     rows[i].forEach((cell: string) => {
    //         const cellElement = document.createElement('td');
    //         cellElement.textContent = cell;
    //         row.appendChild(cellElement);
    //     });
        
    //     // Append the row to the table
    //     tableBody.appendChild(row);
    // }
  } catch (error) {
      console.error('Error fetching Google Sheets data:', error);
  }
}

function getStep(step: Array<string>): Step {
  return {
    'prompt': step[0],
    'dayNight': step[1],
    'tags': step[2],
    'includeOne': step[3],
  }
}

// function dayNightLabel(dayNight: string): string {
//   if (['Day', 'Night'].indexOf(dayNight) !== -1) {
//     return dayNight
//   } else if (dayNight === 'Night') {
//     return 'Night'
//   } else {
//     return ''
//   }
// }

defineOptions({
  name: 'IndexPage'
});

onMounted(() => { 
  // fetchGoogleSheetData()
})
</script>
