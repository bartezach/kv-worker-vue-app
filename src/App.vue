<template>
  <div>
    <h1>Player Config Manager</h1>
    <RolloutTable />
    <WhitelistTable />
  </div>
</template>

<script>
import RolloutTable from "./components/RolloutTable.vue";
import WhitelistTable from "./components/WhitelistTable.vue";

export default {
  components: {
    RolloutTable,
    WhitelistTable
  }
};
</script>

<style>
.app-container {
  padding: 20px;
}

h1 {
  font-family: 'Segoe UI', 'Roboto', 'Open Sans', Arial, sans-serif;
  font-size: 2rem;
  font-weight: 600;
  color: #333;
  text-align: center;
  margin-bottom: 20px;
  position: relative;
}

h1::after {
  content: '';
  display: block;
  width: 60px;
  height: 4px;
  background-color: #4CAF50;
  margin: 8px auto 0;
  border-radius: 2px;
}

h2 {
  font-family: 'Segoe UI', 'Roboto', 'Open Sans', Arial, sans-serif;
  font-size: 1.5rem;
  font-weight: 600;
  color: #333;
  margin-bottom: 10px;
}

h3 {
  font-family: 'Segoe UI', 'Roboto', 'Open Sans', Arial, sans-serif;
  font-size: 1rem;
  font-weight: 600;
  color: #333;
  margin-bottom: 10px;
}

table {
  border-collapse: collapse;
  width: 100%;
  font-family: Arial, sans-serif;
  background-color: #fff;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  margin: 20px 0;
}

th {
  font-family: 'Segoe UI', 'Roboto', 'Open Sans', Arial, sans-serif;
  font-size: 1rem;
  background-color: #4CAF50;
  color: white;
  padding: 14px 12px;
  text-align: left;
}

th, td {
  padding: 12px 12px;
  line-height: 1.6;
  border: 1px solid #ccc;
  vertical-align: middle;
}

tr:nth-child(even) {
  background-color: #f9f9f9;
}

input[readonly] {
  background-color: #f0f0f0;
}

textarea {
  width: 100%;
  min-height: 40px;
  resize: vertical;
  padding: 6px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-family: inherit;
  font-size: 14px;
  line-height: 1.4;
  box-sizing: border-box;
}

button {
  background-color: #4CAF50;
  color: white;
  padding: 6px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  height: 36px;
}

button:hover {
  background-color: #45a049;
}

.update-cell {
  text-align: center;
}

/* Spinner */
.loader {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 24px;
  height: 24px;
  animation: spin 1s linear infinite;
  margin: auto;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>
