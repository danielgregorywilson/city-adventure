<!-- https://medium.com/@ravipatel.it/step-by-step-guide-reading-public-google-sheets-data-using-javascript-and-displaying-it-on-an-html-f6aee8416a9c -->

<template>
  <q-page class="row items-center justify-evenly">
    <q-stepper
      v-model="stepIdx"
      vertical
      color="primary"
      animated
    >
      <q-step
        v-for="step in steps"
        :key="steps.indexOf(step)"
        :name="steps.indexOf(step)"
        :title="step[0]"
        icon="settings"
        :done="stepIdx > steps.indexOf(step)"
      >
        {{ step[1] }}
        <q-stepper-navigation>
          <q-btn @click="stepIdx += 1" color="primary" label="Continue" />
        </q-stepper-navigation>
      </q-step>
    </q-stepper>
  </q-page>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';

const spreadsheetId = '1AbZ0UlsIBKXt6AIvyQDCqasNZTZK-TXD70drcxrsI-8'

const apiKey = import.meta.env.VITE_GOOGLE_API_KEY;

const url = `https://sheets.googleapis.com/v4/spreadsheets/${spreadsheetId}/values/Sheet1?key=${apiKey}`;

let steps = ref([])
let stepIdx = ref(0)

async function fetchGoogleSheetData() {
  try {
    // Fetch data from Google Sheets API
    const response = await fetch(url);
    const data = await response.json();
    
    const rows = data.values; // Extract rows from the data
    rows.shift(); // Remove the header row
    const shuffledRows = rows.sort(() => Math.random() - 0.5)

    // Select 5 from the shuffled rows
    steps.value = shuffledRows.slice(0, 5)

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

defineOptions({
  name: 'IndexPage'
});

onMounted(() => { 
  fetchGoogleSheetData()
})
</script>
