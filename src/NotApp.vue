<script setup></script>

<template>
  <div class="p-6 max-w-4xl mx-auto">
    <h1 class="text-2xl font-bold mb-6">Platform Rollouts</h1>

    <table class="w-full border-collapse">
      <thead>
        <tr class="border-b">
          <th class="text-left py-2">Platform</th>
          <th class="text-left py-2">Rollout %</th>
          <th class="text-left py-2">Comment</th>
          <th class="py-2">Update</th>
        </tr>
      </thead>
      <tbody>
        <!-- Existing KV entries -->
        <tr
          v-for="(entry, index) in entries"
          :key="entry.platform"
          class="border-b"
        >
          <td class="py-2">{{ entry.platform }}</td>
          <td class="py-2">
            <input
              type="number"
              class="border rounded px-2 py-1 w-24"
              v-model.number="entry.rollout"
              min="0"
              max="100"
            />
          </td>
          <td class="py-2">
            <input
              type="text"
              class="border rounded px-2 py-1 w-full"
              v-model="entry.comment"
            />
          </td>
          <td class="py-2 text-center">
            <button
              class="bg-blue-600 hover:bg-blue-700 text-white px-3 py-1 rounded"
              @click="updateEntry(entry)"
            >
              Update
            </button>
          </td>
        </tr>

        <!-- Row for adding new entry -->
        <tr>
          <td class="py-2">
            <input
              type="text"
              class="border rounded px-2 py-1 w-full"
              v-model="newEntry.platform"
              placeholder="New platform"
            />
          </td>
          <td class="py-2">
            <input
              type="number"
              class="border rounded px-2 py-1 w-24"
              v-model.number="newEntry.rollout"
              min="0"
              max="100"
            />
          </td>
          <td class="py-2">
            <input
              type="text"
              class="border rounded px-2 py-1 w-full"
              v-model="newEntry.comment"
              placeholder="Optional comment"
            />
          </td>
          <td class="py-2 text-center">
            <button
              class="bg-green-600 hover:bg-green-700 text-white px-3 py-1 rounded"
              @click="addEntry"
            >
              Add
            </button>
          </td>
        </tr>
      </tbody>
    </table>

    <p v-if="statusMessage" class="mt-4 text-green-700 font-medium">
      {{ statusMessage }}
    </p>
  </div>
</template>

<style scoped></style>
