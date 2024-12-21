<!-- https://medium.com/@ravipatel.it/step-by-step-guide-reading-public-google-sheets-data-using-javascript-and-displaying-it-on-an-html-f6aee8416a9c -->

<template>
  <q-page class="row items-center justify-evenly">
    <table id="data-table">
        <thead>
            <tr>
                <th>Name</th>
                <th>Age</th>
                <th>City</th>
            </tr>
        </thead>
        <tbody>
        </tbody>
    </table>
    
    <example-component
      title="Example component"
      active
      :todos="todos"
      :meta="meta"
    ></example-component>
  </q-page>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
import { Todo, Meta } from 'components/models';
import ExampleComponent from 'components/ExampleComponent.vue';

const spreadsheetId = '1AbZ0UlsIBKXt6AIvyQDCqasNZTZK-TXD70drcxrsI-8'

const apiKey = 'AIzaSyBE8BMLjek5n_8Jp1x4oFWWXWlxW6cO3h4';

const url = `https://sheets.googleapis.com/v4/spreadsheets/${spreadsheetId}/values/Sheet1?key=${apiKey}`;

async function fetchGoogleSheetData() {
  try {
    // Fetch data from Google Sheets API
    const response = await fetch(url);
    const data = await response.json();
    
    // Extract rows from the data
    const rows = data.values;

    // Get the table body element
    const tableBody = document.querySelector('#data-table tbody');
    if (!tableBody) {
        throw new Error('Table body element not found');
    }

    // Loop through the rows (starting from row 1 to skip headers)
    for (let i = 1; i < rows.length; i++) {
        const row = document.createElement('tr');
        
        // Loop through each cell in the row and create a table cell for each
        rows[i].forEach((cell: string) => {
            const cellElement = document.createElement('td');
            cellElement.textContent = cell;
            row.appendChild(cellElement);
        });
        
        // Append the row to the table
        tableBody.appendChild(row);
    }
  } catch (error) {
      console.error('Error fetching Google Sheets data:', error);
  }
}

defineOptions({
  name: 'IndexPage'
});

const todos = ref<Todo[]>([
  {
    id: 1,
    content: 'ct1'
  },
  {
    id: 2,
    content: 'ct2'
  },
  {
    id: 3,
    content: 'ct3'
  },
  {
    id: 4,
    content: 'ct4'
  },
  {
    id: 5,
    content: 'ct5'
  }
]);

const meta = ref<Meta>({
  totalCount: 1200
});

onMounted(() => { 
  fetchGoogleSheetData()
})
</script>
