// src/App.vue
<script setup>
import { ref, onMounted } from 'vue';

const tickets = ref([
  { id: 1, title: 'Fix login bug', description: 'Users cannot login with correct credentials', status: 'Approved' },
  { id: 2, title: 'Add dark mode', description: 'Implement dark mode for better user experience', status: 'Approved' },
  { id: 3, title: 'Update documentation', description: 'Add new features to the docs', status: 'Committed' },
  { id: 4, title: 'Improve performance', description: 'Optimize loading times', status: 'Committed' },
  { id: 5, title: 'Deploy version 1.2', description: 'Release the latest version to production', status: 'Done' },
]);

const newTicket = ref({
  title: '',
  description: ''
});

const columns = ['Approved', 'Committed', 'Done'];
let nextId = 6;

const addTicket = () => {
  if (newTicket.value.title.trim() === '') return;
  
  tickets.value.push({
    id: nextId++,
    title: newTicket.value.title,
    description: newTicket.value.description,
    status: 'Approved' // New tickets always go to Approved column
  });
  
  // Reset form
  newTicket.value.title = '';
  newTicket.value.description = '';
};

// Enhanced drag and drop functionality
const draggedTicket = ref(null);

const startDrag = (event, ticket) => {
  draggedTicket.value = ticket;
  event.dataTransfer.effectAllowed = 'move';
  // Set dragged ticket data
  event.dataTransfer.setData('ticketId', ticket.id);
  
  // Add a drag visual effect
  setTimeout(() => {
    event.target.classList.add('dragging');
  }, 0);
};

const endDrag = (event) => {
  // Remove visual effect
  event.target.classList.remove('dragging');
};

const onDragOver = (event) => {
  event.preventDefault();
  event.dataTransfer.dropEffect = 'move';
};

const onDrop = (event, column) => {
  event.preventDefault();
  const ticketId = parseInt(event.dataTransfer.getData('ticketId'));
  if (ticketId) {
    const ticketIndex = tickets.value.findIndex(t => t.id === ticketId);
    if (ticketIndex !== -1) {
      tickets.value[ticketIndex].status = column;
    }
  }
  
  // Reset the draggedTicket after drop
  draggedTicket.value = null;
};

// Save to localStorage
const saveToLocalStorage = () => {
  localStorage.setItem('tickets', JSON.stringify(tickets.value));
};

// Load from localStorage
const loadFromLocalStorage = () => {
  const savedTickets = localStorage.getItem('tickets');
  if (savedTickets) {
    tickets.value = JSON.parse(savedTickets);
    // Update nextId to be greater than the highest existing id
    nextId = Math.max(...tickets.value.map(t => t.id)) + 1;
  }
};

// Watch for changes and save
onMounted(() => {
  loadFromLocalStorage();
  
  // Set up a watcher for tickets
  watch(tickets, () => {
    saveToLocalStorage();
  }, { deep: true });
});

// Add the missing watch import
import { watch } from 'vue';
</script>

<template>
  <div class="ticket-board">
    <h1>Ticket Management Board</h1>
    
    <!-- New Ticket Form -->
    <div class="new-ticket-form">
      <h2>Create New Ticket</h2>
      <div class="form-group">
        <label for="title">Title:</label>
        <input id="title" v-model="newTicket.title" placeholder="Enter ticket title">
      </div>
      <div class="form-group">
        <label for="description">Description:</label>
        <textarea id="description" v-model="newTicket.description" placeholder="Enter ticket description"></textarea>
      </div>
      <button @click="addTicket">Add Ticket</button>
    </div>
    
    <!-- Board -->
    <div class="board">
      <div 
        v-for="column in columns" 
        :key="column" 
        class="column"
        @dragover="onDragOver"
        @drop="onDrop($event, column)"
      >
        <h2>{{ column }}</h2>
        <div class="ticket-container">
          <div 
            v-for="ticket in tickets.filter(t => t.status === column)" 
            :key="ticket.id" 
            class="ticket"
            draggable="true"
            @dragstart="startDrag($event, ticket)"
            @dragend="endDrag($event)"
          >
            <h3>{{ ticket.title }}</h3>
            <p>{{ ticket.description }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
.ticket-board {
  font-family: Arial, sans-serif;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

h1 {
  text-align: center;
  color: #333;
}

.new-ticket-form {
  background-color: #f5f5f5;
  padding: 20px;
  border-radius: 5px;
  margin-bottom: 20px;
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

input, textarea {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
}

textarea {
  height: 80px;
  resize: vertical;
}

button {
  background-color: #4CAF50;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

button:hover {
  background-color: #45a049;
}

.board {
  display: flex;
  gap: 20px;
  overflow-x: auto;
}

.column {
  background-color: #f5f5f5;
  border-radius: 5px;
  min-width: 300px;
  flex: 1;
  padding: 15px;
}

.column h2 {
  text-align: center;
  margin-top: 0;
  padding-bottom: 10px;
  border-bottom: 2px solid #ddd;
}

.ticket-container {
  min-height: 200px;
}

.ticket {
  background-color: white;
  padding: 15px;
  border-radius: 4px;
  margin-bottom: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  cursor: grab;
  transition: all 0.2s ease;
}

.ticket:hover {
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
}

.ticket.dragging {
  opacity: 0.5;
  transform: scale(0.95);
}

.ticket h3 {
  margin-top: 0;
  margin-bottom: 10px;
  color: #333;
}

.ticket p {
  margin: 0;
  color: #666;
}
</style>