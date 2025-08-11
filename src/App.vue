<template>
  <div>
    <h1>Platform Rollouts</h1>

    <!-- Error message -->
    <div v-if="errorMessage" style="color: red; font-weight: bold;">
      {{ errorMessage }}
    </div>

    <div v-if="isLoading" class="loader"></div>

    <table v-if="!isLoading">
      <thead>
        <tr>
          <th>Platform</th>
          <th>Rollout %</th>
          <th>Comment</th>
          <th>Update</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(entry, key) in kvData" :key="key">
          <td><input type="text" v-model="entry.platform" readonly /></td>
          <td>
            <input type="number" v-model.number="entry.rollout" min="0" max="100" />
          </td>
          <td>
            <textarea v-model="entry.comment" rows="2"></textarea>
          </td>
          <td class="update-cell">
            <button @click="updateEntry(key, entry)">Update</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      kvData: {},
      errorMessage: "",
      isLoading: false
    };
  },
  mounted() {
    this.fetchData();
  },
  methods: {
    async fetchData() {
      this.errorMessage = "";
      this.isLoading = true;
      try {
        const res = await fetch("https://www.rte.ie/dev/player_rollouts/");
        if (!res.ok) throw new Error(`GET failed: ${res.statusText}`);
        const data = await res.json();
        this.kvData = {};

        // Convert KV entries to table
        for (const [key, value] of Object.entries(data.message)) {
          this.kvData[key] = {
            platform: key,
            rollout: value.rollout ? value.rollout * 100 : 0,
            comment: value.comment || ""
          };
        }
      } catch (err) {
        this.errorMessage = err.message;
      } finally {
        this.isLoading = false;
      }
    },

    async updateEntry(key, entry) {
      this.errorMessage = "";

      // Validate rollout value
      if (entry.rollout === null || entry.rollout < 0 || entry.rollout > 100) {
        this.errorMessage = "Rollout % must be between 0 and 100.";
        return;
      }

      try {
        const body = {
          key,
          value: {
            rollout: String(entry.rollout / 100), // Store as 0–1
            comment: entry.comment
          }
        };

        const res = await fetch("https://www.rte.ie/dev/player_rollouts/", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(body)
        });

        if (!res.ok) throw new Error(`POST failed: ${res.statusText}`);
        await this.fetchData(); // Refresh table
      } catch (err) {
        this.errorMessage = err.message;
      }
    }
  }
};
</script>

<style>
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

table {
  border-collapse: collapse;
  width: 100%;
  font-family: Arial, sans-serif;
  background-color: #fff;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

th {
  background-color: #4CAF50;
  color: white;
  padding: 12px;
  text-align: left;
}

th, td {
  padding: 8px;
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
  text-align: center; /* simpler than flexbox */
}

/* Spinner styles */
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
