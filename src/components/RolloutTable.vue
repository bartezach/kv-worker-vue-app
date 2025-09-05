<template>
  <div>
    <h2>Platform Rollouts</h2>

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
        const res = await fetch("https://www.rte.ie/dev/player_rollouts/rollouts");
        if (!res.ok) throw new Error(`GET failed: ${res.statusText}`);
        const data = await res.json();
        this.kvData = {};

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

      if (entry.rollout === null || entry.rollout < 0 || entry.rollout > 100) {
        this.errorMessage = "Rollout % must be between 0 and 100.";
        return;
      }

      try {
        const body = {
          key,
          value: {
            rollout: String(entry.rollout / 100),
            comment: entry.comment
          }
        };

        const res = await fetch("https://www.rte.ie/dev/player_rollouts/rollouts", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(body)
        });

        if (!res.ok) throw new Error(`POST failed: ${res.statusText}`);
        await this.fetchData();
      } catch (err) {
        this.errorMessage = err.message;
      }
    }
  }
};
</script>
