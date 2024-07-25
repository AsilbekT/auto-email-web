<template>
  <div class="min-h-screen flex items-center justify-center bg-gray-100">
    <div class="container mx-auto p-6 bg-white rounded-lg shadow-lg max-w-4xl">
      <h1
        class="text-3xl sm:text-2xl md:text-3xl lg:text-4xl font-bold text-center text-gray-800 mb-8"
      >
        Telegram Message Search
      </h1>
      <div class="mb-6 relative w-full">
        <input
          v-model="loadNumber"
          @input="fetchMessages"
          type="text"
          placeholder="Load Number ..."
          aria-label="Search messages"
          class="w-full pl-10 pr-4 py-2 border rounded-lg text-gray-700 focus:outline-none focus:border-blue-500"
        />
        <div
          class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none"
        >
          <i class="fas fa-search text-gray-500"></i>
        </div>
      </div>
      <div v-if="loading" class="text-center text-blue-500">Loading...</div>
      <div
        v-if="!loading && messages && messages.length === 0"
        class="text-center text-red-500"
      >
        No messages found.
      </div>
      <ul v-if="!loading && messages && messages.length > 0" class="space-y-4">
        <li
          v-for="message in messages"
          :key="message['Message ID']"
          class="p-4 bg-gray-50 rounded-lg shadow border-l-4 border-gray-300 hover:border-blue-500 transition"
          @click="select(message['Message ID'])"
          :class="{ 'border-blue-500': message['Message ID'] == id }"
        >
          <div
            class="flex flex-col md:flex-row justify-between items-center gap-4"
          >
            <div
              class="flex flex-col gap-2 text-center sm:text-center md:text-center lg:text-left"
            >
              <p><strong>From:</strong> {{ message.From }}</p>
              <p><strong>Subject:</strong> {{ message.Subject }}</p>
            </div>
            <select
              class="border bg-white rounded p-2 px-8 shadow cursor-pointer hover:bg-gray-100 transition"
              @change="handleAction($event, message['Message ID'])"
            >
              <option value="">Actions</option>
              <option value="assign">Assign</option>
            </select>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      loadNumber: "",
      messages: [],
      loading: false,
      timer: null,
      id: "",
    };
  },
  methods: {
    async fetchMessages() {
      clearTimeout(this.timer);
      this.timer = setTimeout(async () => {
        if (this.loadNumber.trim() === "") {
          this.messages = [];
          return;
        }
        this.loading = true;
        try {
          const response = await axios.get(
            `http://127.0.0.1:8000/messages?user_principal_name=admin@bosscargollc.com&load_number=${this.loadNumber}`
          );
          this.messages = response.data.messages || [];
          console.log(this.messages);
        } catch (error) {
          console.error("Failed to fetch messages:", error);
          this.messages = [];
        } finally {
          this.loading = false;
        }
      }, 500);
    },
    select(id) {
      this.id = id;
    },
    handleAction(event, messageId) {
      const action = event.target.value;
      if (action === 'assign') {
        this.assignMessage(messageId, this.loadNumber);
        event.target.selectedIndex = 0;
      }
    },
    async assignMessage(messageId, load_number) {
      try {
          const TRACKING_URL = "https://tracking.prisunion.uz/telegram_bot/assign/"
          const response = await axios.post(TRACKING_URL, {"message_id": messageId, "load_number": load_number});
          this.messages = response.data.messages || [];
          alert(`Assigned load number#${load_number} to email ✅`)
          this.loadNumber = ""
        } catch (error) {
          alert(`Load number ${load_number} could not be assigned.`)
          console.error("Failed to assign message:", error);
          this.messages = [];
        } finally {
          this.loading = false;
        }
    }
  },
};
</script>

<style scoped>
input[type="text"]:focus {
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.5);
  transition: box-shadow 0.3s ease-in-out;
}
.active {
  border: #60a5fa 2px solid;
}
select {
  transition: background-color 0.3s ease;
}

select:hover {
  background-color: #f3f4f6;
}

button {
  transition: background-color 0.2s ease, transform 0.1s ease;
}

button:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

button:active {
  transform: translateY(1px);
  box-shadow: 0 2px 3px rgba(0, 0, 0, 0.1);
}

.loading-text {
  animation: pulse 1.5s infinite ease-in-out;
}

li {
  border-left: 4px solid #cbd5e1;
  transition: border-color 0.3s ease;
}

li:hover {
  border-color: #60a5fa;
}

strong {
  color: #374151;
  font-weight: 600;
}
</style>