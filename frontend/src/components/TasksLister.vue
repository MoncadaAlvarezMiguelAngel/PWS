<template>
  <div v-if="checkIfInRole(user, [ 0, 1 ])">
    <v-card variant="text">
      <v-card-title>Tasks</v-card-title>
      <v-card-subtitle>
        <v-container>
          <v-row>
            <v-col>
              <v-text-field variant="solo" label="Search" v-model="search" @input="retrieve"></v-text-field>
            </v-col>
            <v-col cols="4">
              <v-select v-model="status" label="Status" :items="[ { value: 0, title: 'preparation' }, { value: 1, title: 'pending' }, { value: 2, title: 'in tests' }, { value: 3, title: 'completed' } ]" chips multiple @update:modelValue="retrieve">
              </v-select>
            </v-col>
            <v-col cols="2">
              <v-text-field variant="solo" type="number" label="Skip" v-model="skip" @input="retrieve"></v-text-field>
            </v-col>
            <v-col cols="2">
              <v-text-field variant="solo" type="number" label="Limit" v-model="limit" @input="retrieve"></v-text-field>
            </v-col>
          </v-row>
        </v-container>
      </v-card-subtitle>
      <v-card-text>
        <v-table density="compact" hover>
          <thead>
            <tr>
              <th class="text-left">
                Name
              </th>
              <th class="text-left">
                Status
              </th>
              <th class="text-left">
                Start date
              </th>
              <th class="text-left">
                End date
              </th>
              <th class="text-left">
                Project
              </th>
              <th class="text-left">
                Workers
              </th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(task, index) in tasks" :key="index" @click="checkIfInRole(user, [ 1 ]) && click(task)">
              <td>{{ task.name }}</td>
              <td><v-chip :color="[ 'orange darken-3', 'blue darken-2', 'yellow darken-3', 'success' ][task.status]">
                  {{ ['Preparation', 'Pending', 'In tests', 'Completed'][task.status] }}
                  </v-chip></td>
              <td class="text-left">{{ new Date(task.startDate).toLocaleDateString() }}</td>
              <td class="text-left">{{ task.endDate ? new Date(task.endDate).toLocaleDateString() : 'No end date established' }}</td>
              <td>
                <!--<v-chip :color="project.color" v-for="(project, pindex) in projects" :key="pindex" 
                @update:model-value="modify(task)"
                @click.stop="">
                  {{ project.shortcut}}
                </v-chip>-->
                <!--<v-chip-group>
                    <v-chip
                      v-for="(project, pindex) in projects"
                      value: project._id
                      :key="pindex"
                      :color="project.color"                   
                      @click="() => modify2(task, project)"
                      >
                      {{ project.shortcut }}
                    </v-chip>
                </v-chip-group>-->
                <v-select  
                  v-model="task.project_id"
                  label="Project"
                  :items= "projects.map(project => ({ value: project._id, title: project.shortcut, props: { subtitle: project.name + ' ' + new Date(project.startDate).toLocaleDateString() } }))"
                  variant="solo"
                  @click.stop=""
                  @update:model-value="modify(task)"                  
                ></v-select>
              </td>
              <td>
                <v-chip v-for="(person, pindex) in task.workers" :key="pindex">
                  {{ person.firstName + ' ' + person.lastName }}
                </v-chip>
              </td>
            </tr>
            <v-row>
              <v-col cols="6" >
                <GantChart />
            </v-col>
            </v-row>

          </tbody>
        </v-table>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn variant="elevated" color="success" @click="add" v-if="checkIfInRole(user, [ 1 ])">Add</v-btn>
      </v-card-actions>
    </v-card>
    <v-dialog v-model="editor" width="50%">
      <TaskEditor :id="id" @dataChanged="retrieve" @cancel="cancel"/>
    </v-dialog>
  </div>
</template>

<script>
import common from '../mixins/common'
import TaskEditor from './TaskEditor.vue'
import GantChart from './GantChart.vue'

export default {
  name: 'TasksLister',
  components: { TaskEditor, GantChart },
  props: [ 'user' ],
  mixins: [ common ],
  methods: {
    
    retrieve() {
      this.id = null
      this.editor = false
      fetch('/task?search=' + this.search + '&status=' + JSON.stringify(this.status) + '&skip=' + this.skip + '&limit=' + this.limit, {
        method: 'GET' })
        .then((res) => {
          res.json()
            .then((data) => {
              this.tasks = data
            })
            .catch(err => console.error(err.message))
        })
        .catch(err => console.error(err.message))
    },
    add() {
      this.id = null
      this.editor = true
    },
    click(row) {
      this.id = row._id
      this.editor = true
    },
    cancel() {
      this.id = null
      this.editor = false
    },

    modify(task) {
      task.workers=[]
      fetch('/task?_id=' + task._id, {
          method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ project_id: task.project_id, workers:task.workers }) })
        .then((res) => {
          res.json()
            .then(() => {
              this.$emit('dataChanged')
            })
            .catch(err => console.error(err.message))
        })
        .catch(err => console.error(err.message))
    },

    modify2(task, project) {
      task.workers=[]
      fetch('/task?_id=' + task._id, {
          method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ project_id: project.project_id, workers:task.workers }) })
        .then((res) => {
          res.json()
            .then(() => {
              this.$emit('dataChanged')
            })
            .catch(err => console.error(err.message))
        })
        .catch(err => console.error(err.message))
    },

    modifyTaskWithProject(task, project) {
    // Llama a tu función modify con la tarea y el proyecto como argumentos
    this.modify(task, project);
  },
    
  },
  data() {
    return {
      editor: false,
      tasks: [],
      id: null,
      search: '',
      status: [ 0, 1, 2, 3 ],
      skip: 0,
      limit: 10,
      projects: [],
      workers: []
    }
  },
  mounted() {
    fetch('/project?limit=1000', { method: 'GET'})
    .then(res => res.json())
    .then(data => this.projects = data)
    this.retrieve()
  }

}
</script>