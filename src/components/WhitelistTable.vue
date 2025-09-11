<template>
  <div>
    <h2>Whitelist</h2>

    <div v-if="errorMessage" style="color: red; font-weight: bold;">
      {{ errorMessage }}
    </div>

    <div v-if="isLoading" class="loader"></div>

    <!-- new entry -->
    <div v-if="!isLoading">
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
            <button @click="deleteEntry(key)">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

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
      lastDeletedKey: null,
      checkInterval: null,
      attempts: 0,
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
          if (value && typeof value === "object") {
            this.kvData[key] = {
              ipv4: value.ipv4 || "",
              ipv6: value.ipv6 || "",
              comment: value.comment || ""
            };
          } 
        }
      } catch (err) {
        this.errorMessage = err.message;
      }
    },

    async addEntry() {
      this.errorMessage = "";

      if (!this.newEntry.ipv4.trim() && !this.newEntry.ipv6.trim()) {
        this.errorMessage = "At least one of IPv4 or IPv6 must be provided.";
        return;
      }

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

      if (!entry.ipv4.trim() && !entry.ipv6.trim()) {
        this.errorMessage = "At least one of IPv4 or IPv6 must be provided.";
        return;
      }

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

    async deleteEntry(key) {
      this.errorMessage = "";
      try {
        this.isLoading = true;
        this.lastDeletedKey = key;

        const res = await fetch("https://www.rte.ie/dev/player_rollouts/whitelist", {
          method: "DELETE",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ key })
        });

        if (!res.ok) throw new Error(`DELETE failed: ${res.statusText}`);
        console.log("DELETE response:", await res.json());

        await this.startVerificationLoop(true);
      } catch (err) {
        this.errorMessage = err.message;
        this.isLoading = false;
      }
    },

    async startVerificationLoop(isDelete = false) {
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

          if (isDelete) {
            if (this.lastDeletedKey && !this.kvData[this.lastDeletedKey]) {
              clearInterval(this.checkInterval);
              this.isLoading = false;
              console.log("KV delete verified:", this.lastDeletedKey);
              this.lastDeletedKey = null;
              return;
            }
          } else {
            if (this.lastSubmittedKey && this.kvData[this.lastSubmittedKey]) {
              clearInterval(this.checkInterval);
              this.isLoading = false;
              console.log("KV update verified:", this.kvData[this.lastSubmittedKey]);
              this.lastSubmittedKey = null; // added this just now
              return;
            }
          }

          if (this.attempts >= 10) {
            clearInterval(this.checkInterval);
            this.isLoading = false;
            this.errorMessage = isDelete
              ? "The KV you deleted is not in storage."
              : "The KV you added is not in storage.";
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

<style scoped>
.update-cell button {
  margin-right: 6px;
}

.update-cell button:last-child {
  margin-right: 0; /* Remove margin for the last button */
}

.update-cell button:last-child {
  background-color: #e74c3c;
}

</style>
