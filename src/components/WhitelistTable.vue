<template>
  <div>
    <h2>Whitelist</h2>

    <div v-if="errorMessage" style="color: red; font-weight: bold;">
      {{ errorMessage }}
    </div>

    <div v-if="isLoading" class="loader"></div>

    <!-- new entry -->
    <div>
      <h3>Add Whitelist Entry</h3>
      <input type="text" v-model="newEntry.ipv4" placeholder="IPv4" />
      <input type="text" v-model="newEntry.ipv6" placeholder="IPv6" />
      <input type="text" v-model="newEntry.comment" placeholder="Comment" />
      <button @click="addEntry">Add</button>
    </div>

    <table v-if="!isLoading">
      <thead>
        <tr>
          <th>IPv4</th>
          <th>IPv6</th>
          <th>Comment</th>
          <th>Update</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(entry, key) in kvData" :key="key">
          <td><input type="text" v-model="entry.ipv4" /></td>
          <td><input type="text" v-model="entry.ipv6" /></td>
          <td><textarea v-model="entry.comment" rows="2"></textarea></td>
          <td class="update-cell">
            <button @click="updateEntry(key, entry)">Update</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<!-- <script>
import * as rangeCheck from "range_check";

export default {
  data() {
    return {
      kvData: {},
      newEntry: { ipv4: "", ipv6: "", comment: "" },
      errorMessage: "",
      isLoading: false,
      lastSubmittedKey: null,
      checkInterval: null,
      attempts: 0
    };
  },
  mounted() {
    this.fetchData();
  },
  methods: {
    isValidIp(ip) {
      if (!ip) return true; // empty is allowed
      return rangeCheck.isIP(ip) || rangeCheck.isRange(ip);
    },

    async fetchData() {
      this.errorMessage = "";
      try {
        const res = await fetch("https://www.rte.ie/dev/player_rollouts/whitelist");
        if (!res.ok) throw new Error(`GET failed: ${res.statusText}`);
        const data = await res.json();

        console.log("Fetched KV data:", data);

        this.kvData = {};
        for (const [key, value] of Object.entries(data.message)) {
          this.kvData[key] = {
            ipv4: value.ipv4 || "",
            ipv6: value.ipv6 || "",
            comment: value.comment || ""
          };
        }
      } catch (err) {
        this.errorMessage = err.message;
      }
    },

    async addEntry() {
      this.errorMessage = "";

      if (!this.isValidIp(this.newEntry.ipv4)) {
        this.errorMessage = "Invalid IPv4 format.";
        return;
      }
      if (!this.isValidIp(this.newEntry.ipv6)) {
        this.errorMessage = "Invalid IPv6 format.";
        return;
      }

      try {
        this.isLoading = true;

        console.log("Sending new entry via POST:", this.newEntry);

        const res = await fetch("https://www.rte.ie/dev/player_rollouts/whitelist", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(this.newEntry)
        });

        if (!res.ok) throw new Error(`POST failed: ${res.statusText}`);
        this.lastSubmittedKey = null; // server generates UUID
        await this.startVerificationLoop();
        this.newEntry = { ipv4: "", ipv6: "", comment: "" };
      } catch (err) {
        this.errorMessage = err.message;
        this.isLoading = false;
      }
    },

    async updateEntry(key, entry) {
      this.errorMessage = "";

      if (!this.isValidIp(entry.ipv4)) {
        this.errorMessage = "Invalid IPv4 format.";
        return;
      }
      if (!this.isValidIp(entry.ipv6)) {
        this.errorMessage = "Invalid IPv6 format.";
        return;
      }

      try {
        this.isLoading = true;
        this.lastSubmittedKey = key;

        const body = {
          key,
          value: {
            ipv4: entry.ipv4,
            ipv6: entry.ipv6,
            comment: entry.comment
          }
        };

        console.log("Sending update via POST:", body);

        const res = await fetch("https://www.rte.ie/dev/player_rollouts/whitelist", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(body)
        });

        if (!res.ok) throw new Error(`POST failed: ${res.statusText}`);
        await this.startVerificationLoop();
      } catch (err) {
        this.errorMessage = err.message;
        this.isLoading = false;
      }
    },

    async startVerificationLoop() {
      if (this.checkInterval) clearInterval(this.checkInterval);
      this.attempts = 0;

      this.checkInterval = setInterval(async () => {
        this.attempts++;
        console.log(`Verification attempt #${this.attempts}`);

        try {
          await this.fetchData();

          if (this.lastSubmittedKey) {
            console.log("Looking for submitted key:", this.lastSubmittedKey);
            console.log("Current KV data keys:", Object.keys(this.kvData));
          }

          if (
            this.lastSubmittedKey &&
            this.kvData[this.lastSubmittedKey]
          ) {
            clearInterval(this.checkInterval);
            this.isLoading = false;
            console.log("KV update verified:", this.kvData[this.lastSubmittedKey]);
            return;
          }

          if (this.attempts >= 10) {
            clearInterval(this.checkInterval);
            this.isLoading = false;
            this.errorMessage = "The KV you added is not in storage.";
            console.warn("Verification failed after 10 attempts.");
          }
        } catch (err) {
          clearInterval(this.checkInterval);
          this.isLoading = false;
          this.errorMessage = "Error verifying KV: " + err.message;
          console.error("Verification loop error:", err);
        }
      }, 5000); // check every 5s
    }
  }
};
</script> -->

<script>
import * as rangeCheck from "range_check";

export default {
  data() {
    return {
      kvData: {},
      newEntry: { ipv4: "", ipv6: "", comment: "" },
      errorMessage: "",
      isLoading: false,
      lastSubmittedKey: null,
      checkInterval: null,
      attempts: 0
    };
  },
  mounted() {
    this.fetchData();
  },
  methods: {
    isValidIp(ip) {
      if (!ip) return true; // empty is allowed
      return rangeCheck.isIP(ip) || rangeCheck.isRange(ip);
    },

    async fetchData() {
      this.errorMessage = "";
      try {
        const res = await fetch("https://www.rte.ie/dev/player_rollouts/whitelist");
        if (!res.ok) throw new Error(`GET failed: ${res.statusText}`);
        const data = await res.json();
        this.kvData = {};

        for (const [key, value] of Object.entries(data.message)) {
          this.kvData[key] = {
            ipv4: value.ipv4 || "",
            ipv6: value.ipv6 || "",
            comment: value.comment || ""
          };
        }
      } catch (err) {
        this.errorMessage = err.message;
      }
    },

    async addEntry() {
      this.errorMessage = "";

      if (!this.isValidIp(this.newEntry.ipv4)) {
        this.errorMessage = "Invalid IPv4 format.";
        return;
      }
      if (!this.isValidIp(this.newEntry.ipv6)) {
        this.errorMessage = "Invalid IPv6 format.";
        return;
      }

      try {
        this.isLoading = true;
        const res = await fetch("https://www.rte.ie/dev/player_rollouts/whitelist", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(this.newEntry)
        });

        if (!res.ok) throw new Error(`POST failed: ${res.statusText}`);

        const data = await res.json();
        // 🔑 capture the UUID the Worker generated
        this.lastSubmittedKey = data.message.stored.key;

        console.log("POST addEntry response:", data);

        await this.startVerificationLoop();
        this.newEntry = { ipv4: "", ipv6: "", comment: "" };
      } catch (err) {
        this.errorMessage = err.message;
        this.isLoading = false;
      }
    },

    async updateEntry(key, entry) {
      this.errorMessage = "";

      if (!this.isValidIp(entry.ipv4)) {
        this.errorMessage = "Invalid IPv4 format.";
        return;
      }
      if (!this.isValidIp(entry.ipv6)) {
        this.errorMessage = "Invalid IPv6 format.";
        return;
      }

      try {
        this.isLoading = true;
        this.lastSubmittedKey = key;

        const body = {
          key,
          value: {
            ipv4: entry.ipv4,
            ipv6: entry.ipv6,
            comment: entry.comment
          }
        };

        const res = await fetch("https://www.rte.ie/dev/player_rollouts/whitelist", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(body)
        });

        if (!res.ok) throw new Error(`POST failed: ${res.statusText}`);

        const data = await res.json();
        console.log("POST updateEntry response:", data);

        await this.startVerificationLoop();
      } catch (err) {
        this.errorMessage = err.message;
        this.isLoading = false;
      }
    },

    async startVerificationLoop() {
      if (this.checkInterval) clearInterval(this.checkInterval);
      this.attempts = 0;

      this.checkInterval = setInterval(async () => {
        this.attempts++;
        try {
          await this.fetchData();

          console.log(
            `Verification attempt ${this.attempts}, looking for key:`,
            this.lastSubmittedKey
          );
          console.log("Current kvData keys:", Object.keys(this.kvData));

          if (
            this.lastSubmittedKey &&
            this.kvData[this.lastSubmittedKey]
          ) {
            clearInterval(this.checkInterval);
            this.isLoading = false;
            console.log("KV update verified:", this.kvData[this.lastSubmittedKey]);
            return;
          }

          if (this.attempts >= 10) {
            clearInterval(this.checkInterval);
            this.isLoading = false;
            this.errorMessage = "The KV you added is not in storage.";
          }
        } catch (err) {
          clearInterval(this.checkInterval);
          this.isLoading = false;
          this.errorMessage = "Error verifying KV: " + err.message;
        }
      }, 3000);
    }
  }
};
</script>
