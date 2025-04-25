// src/App.vue
<script setup>
import { ref, onMounted } from 'vue';

const tickets = ref([
  { id: 1, title: 'Fix login bug', description: 'Users cannot login with correct credentials', status: 'Approved', priority: 'High' },
  { id: 2, title: 'Add dark mode', description: 'Implement dark mode for better user experience', status: 'Approved', priority: 'Medium' },
  { id: 3, title: 'Update documentation', description: 'Add new features to the docs', status: 'Committed', priority: 'Low' },
  { id: 4, title: 'Improve performance', description: 'Optimize loading times', status: 'Committed', priority: 'High' },
  { id: 5, title: 'Deploy version 1.2', description: 'Release the latest version to production', status: 'Done', priority: 'Medium' },
]);

const newTicket = ref({
  title: '',
  description: '',
  priority: 'Medium'
});

const columns = ['Approved', 'Committed', 'Done'];
const priorities = ['Low', 'Medium', 'High'];
const priorityColors = {
  'Low': '#8BC34A',   // Green
  'Medium': '#FF9800', // Orange
  'High': '#F44336'    // Red
};

let nextId = 6;
const showForm = ref(false);

const addTicket = () => {
  if (newTicket.value.title.trim() === '') return;
  
  tickets.value.push({
    id: nextId++,
    title: newTicket.value.title,
    description: newTicket.value.description,
    status: 'Approved', // New tickets always go to Approved column
    priority: newTicket.value.priority
  });
  
  // Reset form
  newTicket.value.title = '';
  newTicket.value.description = '';
  newTicket.value.priority = 'Medium';
  
  // Hide form after submission
  showForm.value = false;
};

// Toggle form visibility
const toggleForm = () => {
  showForm.value = !showForm.value;
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
  
  // Add visual cue for drop zone
  if (event.currentTarget.classList.contains('column')) {
    event.currentTarget.classList.add('drop-zone-active');
  }
};

const onDragLeave = (event) => {
  // Remove visual cue when leaving drop zone
  if (event.currentTarget.classList.contains('column')) {
    event.currentTarget.classList.remove('drop-zone-active');
  }
};

const onDrop = (event, column) => {
  event.preventDefault();
  
  // Remove visual drop zone indicator
  if (event.currentTarget.classList.contains('column')) {
    event.currentTarget.classList.remove('drop-zone-active');
  }
  
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
});

// Add the watch import
import { watch } from 'vue';

// Watch for changes in tickets array
watch(tickets, () => {
  saveToLocalStorage();
}, { deep: true });

// Calculate column stats
const getColumnStats = (column) => {
  const columnTickets = tickets.value.filter(t => t.status === column);
  return {
    total: columnTickets.length,
    high: columnTickets.filter(t => t.priority === 'High').length,
    medium: columnTickets.filter(t => t.priority === 'Medium').length,
    low: columnTickets.filter(t => t.priority === 'Low').length
  };
};

// Format date
const formatDate = () => {
  const now = new Date();
  return now.toLocaleDateString('en-US', { 
    weekday: 'long', 
    year: 'numeric', 
    month: 'long', 
    day: 'numeric' 
  });
};
</script>

<template>
  <div class="ticket-board">
    <header class="app-header">
      <div class="header-content">
        <h1><span class="icon">🎫</span> Ticket Management Board</h1>
        <div class="date">{{ formatDate() }}</div>
      </div>
      <button class="primary-button add-ticket-btn" @click="toggleForm">
        <span v-if="!showForm">+ New Ticket</span>
        <span v-else>Cancel</span>
      </button>
    </header>
    
    <!-- New Ticket Form (conditionally displayed) -->
    <div class="new-ticket-form" v-if="showForm">
      <div class="form-header">
        <h2>Create New Ticket</h2>
      </div>
      <div class="form-body">
        <div class="form-group">
          <label for="title">Title</label>
          <input id="title" v-model="newTicket.title" placeholder="Enter ticket title">
        </div>
        <div class="form-group">
          <label for="description">Description</label>
          <textarea id="description" v-model="newTicket.description" placeholder="Enter ticket description"></textarea>
        </div>
        <div class="form-group">
          <label for="priority">Priority</label>
          <select id="priority" v-model="newTicket.priority">
            <option v-for="priority in priorities" :key="priority" :value="priority">{{ priority }}</option>
          </select>
        </div>
        <div class="form-actions">
          <button class="cancel-button" @click="toggleForm">Cancel</button>
          <button class="submit-button" @click="addTicket">Create Ticket</button>
        </div>
      </div>
    </div>
    
    <!-- Board -->
    <div class="board">
      <div 
        v-for="column in columns" 
        :key="column" 
        class="column"
        @dragover="onDragOver"
        @dragleave="onDragLeave"
        @drop="onDrop($event, column)"
      >
        <div class="column-header" :class="`column-${column.toLowerCase()}`">
          <h2>{{ column }}</h2>
          <div class="column-stats">
            <span class="stat-badge">{{ getColumnStats(column).total }}</span>
          </div>
        </div>
        
        <div class="ticket-container">
          <div 
            v-for="ticket in tickets.filter(t => t.status === column)" 
            :key="ticket.id" 
            class="ticket"
            draggable="true"
            @dragstart="startDrag($event, ticket)"
            @dragend="endDrag($event)"
          >
            <div class="ticket-header">
              <div class="priority-indicator" :style="{ backgroundColor: priorityColors[ticket.priority] }">
                {{ ticket.priority }}
              </div>
              <div class="ticket-id">#{{ ticket.id }}</div>
            </div>
            <h3>{{ ticket.title }}</h3>
            <p>{{ ticket.description }}</p>
          </div>
          
          <!-- Empty column placeholder -->
          <div v-if="!tickets.filter(t => t.status === column).length" class="empty-column">
            <div class="empty-message">No tickets in this column</div>
            <div class="empty-icon">📋</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
:root {
  --primary-color: #4361ee;
  --secondary-color: #3f37c9;
  --success-color: #4caf50;
  --danger-color: #f44336;
  --text-color: #333;
  --text-light: #666;
  --bg-color: #f9fafb;
  --card-bg: #fff;
  --border-color: #e0e0e0;
  --approved-color: #3a86ff;
  --committed-color: #8338ec;
  --done-color: #06d6a0;
  --header-bg: #fff;
  --shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  --hover-shadow: 0 10px 15px rgba(0, 0, 0, 0.1);
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: var(--bg-color);
  color: var(--text-color);
  line-height: 1.6;
}

.ticket-board {
  max-width: 1400px;
  margin: 0 auto;
  padding: 20px;
}

.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 0;
  margin-bottom: 20px;
  border-bottom: 1px solid var(--border-color);
}

.header-content {
  display: flex;
  flex-direction: column;
}

.app-header h1 {
  margin: 0;
  font-size: 1.8rem;
  color: var(--primary-color);
  display: flex;
  align-items: center;
}

.icon {
  margin-right: 8px;
}

.date {
  font-size: 0.9rem;
  color: var(--text-light);
  margin-top: 4px;
}

.primary-button {
  background-color: var(--primary-color);
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 500;
  transition: all 0.3s ease;
}

.primary-button:hover {
  background-color: var(--secondary-color);
  transform: translateY(-2px);
}

.new-ticket-form {
  background-color: var(--card-bg);
  border-radius: 8px;
  margin-bottom: 25px;
  box-shadow: var(--shadow);
  overflow: hidden;
}

.form-header {
  background-color: var(--primary-color);
  color: white;
  padding: 15px 20px;
}

.form-header h2 {
  margin: 0;
  font-size: 1.3rem;
}

.form-body {
  padding: 20px;
}

.form-group {
  margin-bottom: 20px;
}

label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
  color: var(--text-color);
}

input, textarea, select {
  width: 100%;
  padding: 12px;
  border: 1px solid var(--border-color);
  border-radius: 6px;
  font-size: 1rem;
  transition: border 0.2s ease;
}

input:focus, textarea:focus, select:focus {
  outline: none;
  border-color: var(--primary-color);
  box-shadow: 0 0 0 2px rgba(67, 97, 238, 0.2);
}

textarea {
  height: 100px;
  resize: vertical;
}

select {
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  background-image: url("data:image/svg+xml;charset=utf-8,%3Csvg xmlns='http://www.w3.org/2000/svg' width='16' height='16' viewBox='0 0 24 24' fill='none' stroke='%23333' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M6 9l6 6 6-6'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 10px center;
  background-size: 16px;
}

.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}

.cancel-button {
  background-color: transparent;
  color: var(--text-color);
  border: 1px solid var(--border-color);
  padding: 10px 20px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 1rem;
  transition: all 0.3s ease;
}

.cancel-button:hover {
  background-color: #f5f5f5;
}

.submit-button {
  background-color: var(--success-color);
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 1rem;
  transition: all 0.3s ease;
}

.submit-button:hover {
  background-color: #3d8b40;
  transform: translateY(-2px);
}

.board {
  display: flex;
  gap: 20px;
  overflow-x: auto;
  padding-bottom: 20px;
}

.column {
  background-color: var(--card-bg);
  border-radius: 8px;
  min-width: 320px;
  flex: 1;
  display: flex;
  flex-direction: column;
  box-shadow: var(--shadow);
  transition: box-shadow 0.3s ease;
  overflow: hidden;
}

.column-header {
  padding: 15px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid var(--border-color);
}

.column-approved {
  border-top: 3px solid var(--approved-color);
}

.column-committed {
  border-top: 3px solid var(--committed-color);
}

.column-done {
  border-top: 3px solid var(--done-color);
}

.column h2 {
  margin: 0;
  font-size: 1.2rem;
  font-weight: 600;
}

.column-stats {
  display: flex;
  align-items: center;
}

.stat-badge {
  background-color: #e9ecef;
  color: var(--text-color);
  padding: 2px 8px;
  border-radius: 12px;
  font-size: 0.85rem;
  font-weight: 600;
}

.ticket-container {
  padding: 15px;
  flex-grow: 1;
  min-height: 300px;
}

.ticket {
  background-color: var(--card-bg);
  padding: 15px;
  border-radius: 6px;
  margin-bottom: 12px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  cursor: grab;
  transition: all 0.2s ease;
  border: 1px solid var(--border-color);
}

.ticket:hover {
  box-shadow: var(--hover-shadow);
  transform: translateY(-2px);
}

.ticket.dragging {
  opacity: 0.6;
  transform: scale(0.95);
  box-shadow: var(--hover-shadow);
}

.ticket-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.priority-indicator {
  color: white;
  padding: 3px 8px;
  border-radius: 4px;
  font-size: 0.75rem;
  font-weight: 600;
}

.ticket-id {
  font-size: 0.8rem;
  color: var(--text-light);
}

.ticket h3 {
  margin: 0 0 8px 0;
  font-size: 1.05rem;
  color: var(--text-color);
}

.ticket p {
  margin: 0;
  color: var(--text-light);
  font-size: 0.9rem;
  line-height: 1.5;
}

.empty-column {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 200px;
  border: 2px dashed #e0e0e0;
  border-radius: 6px;
  color: var(--text-light);
}

.empty-message {
  font-size: 0.9rem;
  margin-bottom: 10px;
}

.empty-icon {
  font-size: 2rem;
  opacity: 0.5;
}

.drop-zone-active {
  background-color: rgba(67, 97, 238, 0.05);
  box-shadow: inset 0 0 0 2px var(--primary-color);
}

@media (max-width: 768px) {
  .board {
    flex-direction: column;
  }
  
  .column {
    min-width: 100%;
  }
  
  .app-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 15px;
  }
  
  .add-ticket-btn {
    width: 100%;
  }
}
</style>