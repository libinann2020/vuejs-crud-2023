<template>
<div class="card text-center">
    <div class="card-body">
        <div class="form-floating mb-4">
            <input type="text" class="form-control" v-model="taskData.title" id="task" required>
            <label for="task">Task Title</label>
            <span class="text-danger" v-show="taskErrors.title">{{ taskErrormsgs.title }}</span>
        </div>
        <div class="form-floating mb-4">
            <textarea class="form-control" v-model="taskData.desc" id="desc" required></textarea>
            <label for="desc">Task Description</label>
            <span class="text-danger" v-show="taskErrors.desc">{{ taskErrormsgs.desc }}</span>
        </div>
        <div class="form-floating mb-4">
            <select class="form-select" v-model="taskData.type" id="type" required>
                <option value="" selected>Select Task Type</option>
                <option value="bug">Bug</option>
                <option value="task">Task</option>
                <option value="epic">Epic</option>
            </select>
            <span class="text-danger" v-show="taskErrors.type">{{ taskErrormsgs.type }}</span>
        </div>
        <div class="form-floating mb-4">
            <input type="date" v-model="taskData.dueDate" class="form-control" id="dueDate" required>
            <label for="dueDate">Due Date</label>
            <span class="text-danger" v-show="taskErrors.dueDate">{{ taskErrormsgs.dueDate }}</span>
        </div>
        <button type="submit" v-if="createMode" @click="storeTask()" class="btn btn-success">
            Create Task
        </button>
        <button type="submit" v-else @click="updateTask()" class="btn btn-success">
            Update Task
        </button>
    </div>
    <div class="card-footer">
        <div class="alert alert-success">
            {{ this.successMsg }}
        </div>
    </div>
</div>
<div class="clearfix"></div>
<div class="table-responsive">
    <table class="table table-bordered">
        <thead>
            <tr class="table-info">
                <th colspan="5">All tasks</th>
            </tr>
            <tr>
                <th scope="col">Title</th>
                <th scope="col">Type</th>
                <th scope="col">Desc</th>
                <th scope="col">Due Date</th>
                <th scope="col">Actions</th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="task in tasks">
                <td>{{ task.title }}</td>
                <td>{{ task.type }}</td>
                <td>{{ task.desc }}</td>
                <td>{{ task.dueDate }}</td>
                <td>
                    <button type="submit" @click="editTask(task)" class="btn btn-success btn-sm me-4">
                        Edit
                    </button>
                    <button class="btn btn-danger btn-sm" @click="deleteConfirmation(task)" data-bs-toggle="modal" data-bs-target="#exampleModal">
                        Delete
                    </button>
                </td>
            </tr>
        </tbody>
    </table>
</div>
<delete-component v-bind:title="taskData.title" @delete-task="deleteTask()"/>
</template>
<script>
import axios from 'axios';

export default {
    data(){
        return{
            successMsg:'',
            tasks:{},
            createMode:true,
            taskData: {
                title:'',
                desc:'',
                type:'',
                dueDate:'',
                id:''
            },
            taskErrors: {
                title:false,
                desc:false,
                type:false,
                dueDate:false
            },
            taskErrormsgs:{
                title:'',
                desc:'',
                type:'',
                dueDate:''
            }
        }
    },
    methods: {
        storeTask() {
            axios.post('tasks', this.taskData)
            .then(response => {
                if(response.status == 200) {
                    this.taskData.title = '';
                    this.taskData.type = '';
                    this.taskData.desc = '';
                    this.taskData.dueDate = '';
                    this.successMsg="Task has been created successfully";
                    this.getTasks();
                }
            }).catch(errors => {
                console.log(errors)
                if(errors.response.data.errors.title) {
                    this.taskErrors.title=true;
                    this.taskErrormsgs.title=errors.response.data.errors.title[0];
                }
                if(errors.response.data.errors.desc) {
                    this.taskErrors.desc=true;
                    this.taskErrormsgs.desc=errors.response.data.errors.desc[0];
                }
                if(errors.response.data.errors.type) {
                    this.taskErrors.type=true;
                    this.taskErrormsgs.type=errors.response.data.errors.type[0];
                }
                if(errors.response.data.errors.dueDate) {
                    this.taskErrors.dueDate=true;
                    this.taskErrormsgs.dueDate=errors.response.data.errors.dueDate[0];
                }
            });
        },
        getTasks() {
            axios.get('tasks')
            .then(response => {
                this.tasks=response.data;
            }).catch(errors => {
                console.log(errors)
            })
        },
        editTask(task) {
            console.log(task)
            this.taskData.title = task.title;
            this.taskData.type = task.type;
            this.taskData.desc = task.desc;
            this.taskData.dueDate = task.dueDate;
            this.taskData.id = task.id;
            this.createMode=false;
        },
        updateTask() {
            axios.put('tasks/'+ this.taskData.id, this.taskData)
            .then(response => {
                if(response.status == 200) {
                    this.taskData.title = '';
                    this.taskData.type = '';
                    this.taskData.desc = '';
                    this.taskData.dueDate = '';
                    this.taskData.id = '';
                    this.createMode=true;
                    this.successMsg="Task has been updated successfully";
                    this.getTasks();
                }
            }).catch(errors => {
                console.log(errors)
                if(errors.response.data.errors.title) {
                    this.taskErrors.title=true;
                    this.taskErrormsgs.title=errors.response.data.errors.title[0];
                }
                if(errors.response.data.errors.desc) {
                    this.taskErrors.desc=true;
                    this.taskErrormsgs.desc=errors.response.data.errors.desc[0];
                }
                if(errors.response.data.errors.type) {
                    this.taskErrors.type=true;
                    this.taskErrormsgs.type=errors.response.data.errors.type[0];
                }
                if(errors.response.data.errors.dueDate) {
                    this.taskErrors.dueDate=true;
                    this.taskErrormsgs.dueDate=errors.response.data.errors.dueDate[0];
                }
            });
        },
        deleteConfirmation(task) {
            this.taskData.id = task.id;
            this.taskData.title = task.title;
        },
        deleteTask() {
            axios.delete('tasks/'+ this.taskData.id, this.taskData)
            .then(response => {
                if(response.status == 200) {
                    this.taskData.title = '';
                    this.taskData.id = '';
                    this.createMode=true;
                    this.successMsg="Task has been deleted successfully";
                    this.getTasks();
                }
            }) 
        }
    },
    mounted() {
        this.getTasks();
    }
}
</script>


