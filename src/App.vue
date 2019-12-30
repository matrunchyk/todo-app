<template>
  <div class="container">
    <div id="app" class="tasks">
      <h2>Task list</h2>
      <div v-if="showDeleteConfirmation" class="delete-modal-overlay">
        <div class="delete-modal">
          <div>
            Are you sure you want to delete task
            <span
              :class="{ complete: deletingTask.checked }"
            >"{{ deletingTask.name }}"?</span>
          </div>
          <div class="buttons">
            <button @click="deleteTask">Yes</button>
            <button @click="showDeleteConfirmation = false">No</button>
          </div>
        </div>
      </div>
      <div class="message">
        <div v-if="!tasks.length">You don't have tasks to do.</div>
        <template v-else-if="!incompletedTasks.length">
          <div>🎊</div>
          <div>All done!</div>
        </template>
      </div>
      <div
        v-for="task in incompletedTasks"
        :key="task.id"
        class="task"
        :class="{ checked: task.checked }"
      >
        <a @click="toggleCompleteness(task)"/>
        <span
          :ref="`task-editor-${task.id}`"
          :contenteditable="task.editing"
          @dblclick="enableEditMode(task)"
          @keydown.enter="saveTask(task)"
        >{{ task.name }}</span>
        <button @click="deleteTaskConfirm(task)">🗑</button>
      </div>

      <input v-model="taskName" placeholder="New task..." class="new-task" @keydown.enter="addTask">

      <div class="summary" v-if="tasks.length">
        <div class="hr"/>
        <div class="stats">
          <span class="complete">{{ completedTasks.length }}</span>/
          <span class="incomplete">{{ tasks.length }}</span> (
          <span class="percentage" :class="percentageClass">{{ percentage }}%</span> done)
        </div>
      </div>

      <div
        v-for="task in completedTasksLimited"
        :key="task.id"
        class="task"
        :class="{ checked: task.checked }"
      >
        <a @click="toggleCompleteness(task)"/>
        <span>{{ task.name }}</span>
        <button @click="deleteTaskConfirm(task)">🗑</button>
      </div>
      <div v-if="completedTasks.length > completedTasksLimited.length" class="show-more">
        <span @click="showMore">Show more...</span>
      </div>
    </div>
    <footer>
      © 2019
      <a href="https://digitalidea.studio/" target="_blank">Digital Idea</a>
    </footer>
  </div>
</template>

<script>
// 💩💩💩💩💩💩💩💩💩💩💩💩💩💩💩💩
// Refactoring needed!
// 💩💩💩💩💩💩💩💩💩💩💩💩💩💩💩💩
const defaultTasks = [
  {
    id: 123,
    name: "Start learning Vue",
    checked: true,
    editing: false
  },
  {
    id: 234,
    name: "Save the World",
    checked: false,
    editing: false
  }
];

export default {
  name: "App",
  data: () => ({
    taskName: "",
    defaultTasks,
    tasks: defaultTasks,
    deletingTask: null,
    showDeleteConfirmation: false,
    showMoreIncrementor: 5
  }),
  computed: {
    sortedTasks() {
      // eslint-disable-next-line
      return this.tasks.sort((a, b) => a.checked - b.checked);
    },
    incompletedTasks() {
      return this.sortedTasks.filter(task => !task.checked);
    },
    completedTasks() {
      return this.sortedTasks.filter(task => task.checked);
    },
    completedTasksLimited() {
      return this.completedTasks.slice(0, this.showMoreIncrementor);
    },
    percentage() {
      return Math.trunc((100 / this.tasks.length) * this.completedTasks.length);
    },
    percentageClass() {
      return {
        red: this.percentage > 0,
        yellow: this.percentage >= 50,
        green: this.percentage > 90
      };
    }
  },
  watch: {
    tasks: {
      handler(tasks) {
        window.location.hash = btoa(encodeURIComponent(JSON.stringify(tasks)));
      },
      deep: true
    }
  },
  mounted() {
    const hash = window.location.hash.substr(1);
    if (hash) {
      this.tasks = JSON.parse(decodeURIComponent(atob(hash)));
    } else if (localStorage.getItem("tasks")) {
      this.tasks = JSON.parse(localStorage.getItem("tasks"));
    }
  },
  methods: {
    addTask() {
      this.tasks.push({
        id: new Date().getTime(),
        name: this.taskName,
        checked: false,
        editing: false
      });
      this.saveTasks();
      this.taskName = "";
    },
    saveTask(task) {
      task.editing = false;
      const [editor] = this.$refs[`task-editor-${task.id}`];
      task.name = editor.textContent;
    },
    deleteTaskConfirm(task) {
      this.deletingTask = task;
      this.showDeleteConfirmation = true;
    },
    deleteTask() {
      this.tasks = this.tasks.filter(task => task.id !== this.deletingTask.id);
      this.saveTasks();
      this.showDeleteConfirmation = false;
    },
    toggleCompleteness(task) {
      task.checked = !task.checked;
      this.saveTasks();
    },
    saveTasks() {
      localStorage.setItem("tasks", JSON.stringify(this.tasks));
    },
    showMore() {
      this.showMoreIncrementor += 5;
    },
    enableEditMode(task) {
      task.editing = true;
      const [editor] = this.$refs[`task-editor-${task.id}`];

      setTimeout(() => {
        editor.focus();

        const selection = window.getSelection();
        const range = document.createRange();

        range.selectNodeContents(editor.firstChild);
        selection.removeAllRanges();
        selection.addRange(range);
      }, 0);
    }
  }
};
</script>

<style>
body,
html {
  margin: 0;
  padding: 0;
}

body {
  background: url(https://images.pexels.com/photos/1034425/pexels-photo-1034425.jpeg?auto=format%2Ccompress&cs=tinysrgb&dpr=2&h=750&w=1260);
  background-size: cover;
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  font-size: 18px;
  padding: 20px;
  position: relative;
  background: #fff;
  opacity: 0.95;
  box-shadow: 1px 1px 2px 1px #aaa;
  margin: 0;
  width: 100%;
}

@media only screen and (min-device-width: 768px) {
  #app {
    margin: 60px auto 30px auto;
    width: 50%;
  }
}

h2 {
  text-align: center;
}

.message {
  padding: 20px;
  display: grid;
  justify-items: center;
  align-items: center;
  line-height: 30px;
  color: grey;
  font-weight: bold;
}

.summary {
  display: grid;
  grid-template-columns: auto max-content;
  font-size: 14px;
  grid-gap: 10px;
  margin: 20px 0;
}

.hr {
  border-bottom: 1px solid #eee;
  height: 10px;
}

.task {
  display: grid;
  grid-template-columns: 40px auto 40px;
  grid-gap: 15px;
  justify-items: baseline;
  align-items: center;
  margin-bottom: 20px;
}

.task a {
  display: block;
  border: 1px solid aquamarine;
  width: 25px;
  height: 25px;
  border-radius: 25px;
  cursor: pointer;
}
.task button {
  display: block;
  border: 1px solid lightpink;
  width: 25px;
  height: 25px;
  padding: 0;
  padding-left: 3px;
  border-radius: 2px;
}

.task a:hover {
  background: #7fffd440;
}
.tasks .task.checked a {
  background: aquamarine;
  border: 1px solid #6cdcb6;
}
.tasks .task.checked span,
.delete-modal .complete {
  text-decoration: line-through;
}

.new-task {
  width: calc(100% - 120px);
  font-size: 18px;
  margin-left: 55px;
  padding: 1px 5px;
  border: none;
  border-bottom: 1px dashed lightgray;
  color: gray;
}

.new-task::placeholder {
  color: lightgray;
}

.new-task:active,
.new-task:focus {
  border-bottom: 1px solid #9a9999;
  outline: none !important;
}

.delete-modal-overlay {
  background: #0000005a;
  z-index: 100;
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
}

.delete-modal {
  position: absolute;
  border: 1px solid lightgray;
  background: #fff;
  padding: 30px;
  display: grid;
  grid-template-rows: auto min-content;
  box-shadow: 0 0 1px 2px #eee;
  left: 22%;
  top: 33%;
  z-index: 101;
}

.buttons {
  margin-top: 20px;
  display: grid;
  grid-template-columns: min-content min-content;
  grid-gap: 20px;
  justify-content: center;
}

.buttons button {
  padding: 5px;
  border: 1px solid lightgray;
  border-radius: 2px;
  min-width: 50px;
  font-size: 16px;
  cursor: pointer;
}
.buttons button:hover {
  background: #efefef;
}
.percentage.red {
  color: orangered;
}
.percentage.yellow {
  color: orange;
}
.percentage.green {
  color: olivedrab;
}
.show-more {
  display: grid;
  justify-content: center;
  font-size: 12px;
  cursor: pointer;
}
.show-more:hover {
  text-decoration: underline;
}
footer {
  position: absolute;
  bottom: 10px;
  right: 10px;
  color: #fff;
  font-size: 12px;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
}
footer a {
  color: #fff;
  font-size: 12px;
}
</style>
