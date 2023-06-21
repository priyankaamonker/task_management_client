<template>
    <div class="container">
      <div class="row">
        <div class="col-sm-10">
          <h1>Tasks</h1>
          <hr><br><br>
          <button type="button" class="btn btn-success btn-sm" @click="toggleAddTaskModal">Add Task</button>
          <br><br>
          <table class="table table-hover">
            <thead>
              <tr>
                <th scope="col">Todo</th>
                <th scope="col">Category</th>
                <th scope="col">Assinged to</th>
                <th scope="col">Deadline</th>
                <th></th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(task, index) in tasks" :key="index">
                <td>{{ task.name }}</td>
                <td>{{ task.category.id }}</td>
                <td>
                    <span v-if="task.user == null">Not Assinged</span>
                    <span v-else>{{ task.user.username }}</span>
                </td>
                <td>{{ task.deadline.substr(0, 10) }}</td>
                <td>
                  <div class="btn-group" role="group">
                    <button type="button" class="btn btn-warning btn-sm">Update</button>
                    <button type="button" class="btn btn-danger btn-sm">Delete</button>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>


    <!-- add new task modal -->
    <div
    ref="addTaskModal"
    class="modal fade"
    :class="{ show: activeAddTaskModal, 'd-block': activeAddTaskModal }"
    tabindex="-1"
    role="dialog">
    <div class="modal-dialog" role="document">
        <div class="modal-content">
        <div class="modal-header">
            <h5 class="modal-title">Add a new task</h5>
            <button
            type="button"
            class="close"
            data-dismiss="modal"
            aria-label="Close"
            @click="toggleAddTaskModal">
            <span aria-hidden="true">&times;</span>
            </button>
        </div>
        <div class="modal-body">
            <form>
            <div class="mb-3">
                <label for="addTaskName" class="form-label">Name:</label>
                <input
                type="text"
                class="form-control"
                id="addTaskName"
                v-model="addTaskForm.name"
                placeholder="Enter name">
            </div>
            <div class="mb-3">
                <label for="addTaskDescription" class="form-label">Description:</label>          
                <textarea 
                class="form-control"
                id="addTaskDescription"
                v-model="addTaskForm.description"
                placeholder="Enter description">
                </textarea>
            </div>
            <div class="mb-3">
                <label for="addTaskCategory" class="form-label">Category:</label>
                <select class="form-control" id="addTaskCategory" v-model="addTaskForm.category_id">
                    <option
                    v-for="option in addTaskForm.options" 
                    :value="option.value" 
                    :disabled="option.disabled"
                    :selected="option.selected"
                    >
                    {{ option.text }} 
                    </option>
                </select>
            </div>
            <div class="mb-3">
                <label for="addTaskDeadline" class="form-label">Deadline:</label>       
                <input 
                type="date"
                class="form-control" 
                id="addTaskDeadline"
                v-model="addTaskForm.deadline"
                placeholder="Enter a deadline"/>   
            </div>
            <div class="mb-3">
                <label for="addTaskPriority" class="form-label">Priority:</label>
                <select class="form-control" id="addTaskPriority" v-model="addTaskForm.priority">
                    <option
                    v-for="option in addTaskForm.priority_options" 
                    :value="option.value" 
                    :disabled="option.disabled"
                    :selected="option.selected"
                    >
                    {{ option.text }} 
                    </option>
                </select>
            </div>
            <div class="mb-3">
                <label for="addTaskProgress" class="form-label">Progress:</label>
                <select class="form-control" id="addTaskProgress" v-model="addTaskForm.progress">
                    <option
                    v-for="option in addTaskForm.progress_options" 
                    :value="option.value" 
                    :disabled="option.disabled"
                    :selected="option.selected"
                    >
                    {{ option.text }} 
                    </option>
                </select>
            </div>
            <div class="mb-3">
                <label for="addTaskUser" class="form-label">Assingee:</label>
                <select class="form-control" id="addTaskUser" v-model="addTaskForm.user_id">
                    <option
                    v-for="option in addTaskForm.user_id_options" 
                    :value="option.value" 
                    :disabled="option.disabled"
                    :selected="option.selected"
                    >
                    {{ option.text }} 
                    </option>
                </select>
            </div>            
            <div class="btn-group" role="group">
                <button
                type="button"
                class="btn btn-primary btn-sm"
                @click="handleAddSubmit">
                Submit
                </button>
                <button
                type="button"
                class="btn btn-danger btn-sm"
                @click="handleAddReset">
                Reset
                </button>
            </div>
            </form>
        </div>
        </div>
    </div>
    </div>
    <div v-if="activeAddTaskModal" class="modal-backdrop fade show"></div>
    </div>
  </template>


<script>
import axios from 'axios';

export default {
  data() {
    return {
      activeAddTaskModal: false,
      addTaskForm: {
        name: '',
        description: '',
        options: [
        { text: 'Please select one', value: "", disabled: true },
        { text: 'One', value: '1' },
        { text: 'Seven', value: '7' },
        { text: 'Eight', value: '8' }
        ],
        deadline: '',
        priority_options: [
        { text: 'Please select one', value: "", disabled: true },
        { text: 'High', value: '1' },
        { text: 'Medium', value: '2' },
        { text: 'Low', value: '3' }
        ],
        progress_options: [
        { text: 'Please select one', value: "", disabled: true },
        { text: '0%', value: '0' },
        { text: '10%', value: '10' },
        { text: '20%', value: '20' },
        { text: '40%', value: '40' },
        { text: '60%', value: '60' },
        { text: '80%', value: '80' },
        { text: '100%', value: '100' }
        ],
        user_id_options: [
        { text: 'Please select one', value: "", disabled: true },
        { text: 'Priyanka', value: '1' },
        { text: 'John', value: '3' },
        { text: 'Don', value: '4' },
        { text: 'Frieda', value: '6' },
        ],
      },
      tasks: [],
    };
  },
  methods: {
    addTask(payload) {
      const path = 'http://127.0.0.1:5000/task';
      axios.post(path, payload)
        .then(() => {
          this.getTasks();
        })
        .catch((error) => {

          console.log(error);
          this.getTasks();
        });
    },
    getTasks() {
      const path = 'http://127.0.0.1:5000/task';
      axios.get(path)
        .then((res) => {
          this.tasks = res.data;
        })
        .catch((error) => {
          console.error(error);
        });
    },
    handleAddReset() {
      this.initForm();
    },
    handleAddSubmit() {
      this.toggleAddTaskModal();
      const payload = {
        name: this.addTaskForm.name,
        description: this.addTaskForm.description,
        category_id: this.addTaskForm.category_id,
        deadline: this.addTaskForm.deadline + " 00:00:00",
        priority: this.addTaskForm.priority,
        progress: this.addTaskForm.progress,
        user_id: this.addTaskForm.user_id
      };
      this.addTask(payload);
      this.initForm();
    },
    initForm() {
      this.addTaskForm.name = '';
      this.addTaskForm.description = '';
      this.addTaskForm.category_id = null;
      this.addTaskForm.deadline = '';
      this.addTaskForm.priority = '';
      this.addTaskForm.progress = 0;
      this.addTaskForm.user_id = null;
    },
    toggleAddTaskModal() {
      const body = document.querySelector('body');
      this.activeAddTaskModal = !this.activeAddTaskModal;
      if (this.activeAddTaskModal) {
        body.classList.add('modal-open');
      } else {
        body.classList.remove('modal-open');
      }
    },
  },
  created() {
    this.getTasks();
  },
};
</script>