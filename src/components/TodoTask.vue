<template>
  <v-container>
    <v-responsive class="align-center text-center">
      <v-app-bar color="primary" density="compact">
        <v-toolbar-title class="flex text-center"><v-btn icon="mdi-menu"></v-btn>FRAMEWORKS</v-toolbar-title>
        <template v-slot:append>
          <v-btn prepend-icon="mdi-plus-circle-outline" variant="tonal" @click="createDialog()">Add</v-btn>
        </template>
      </v-app-bar>
      <v-main>
        <v-table density="compact">
          <thead> 
            <tr>
              <th class="text-center">Title</th>
              <th class="text-center">Description</th>
              <th class="text-center">Deadline</th>
              <th class="text-center">Priority</th>
              <th class="text-center">Is Complete</th>
              <th class="text-center">Action</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in todolist">
              <td class="text-center">{{ item.title }}</td>
              <td class="text-center">{{ item.description }}</td>
              <td class="text-center">{{ formatDate(item.deadline) }}</td>
              <td class="text-center">{{ item.priority }}</td>
              <td class="text-center">
                <v-checkbox v-model="item.iscomplete" class="d-inline-flex" hide-details></v-checkbox>
              </td>
              <td class="text-center">
                <v-btn variant="elevated" prepend-icon="mdi-square-edit-outline" size="small" color="primary"
                  @click="updateDialog(item)" v-show="item.iscomplete == false">UPDATE</v-btn>
                <v-btn variant="elevated" prepend-icon="mdi-close-circle-outline" size="small" color="red"
                  @click="deleteItem(item)">DELETE</v-btn>
              </td>
            </tr>
          </tbody>
        </v-table>
      </v-main>
      <!-- Add Dialog -->
      <v-dialog v-model="taskDialog" persistent max-width="300px">
        <v-card>
          <v-form ref="taskform" @submit.prevent>
            <v-card-title><v-icon size="x-small">{{ taskDialogTitleIcon }}</v-icon>
              {{ taskDialogTitle }}</v-card-title>
            <v-card-text>
              <v-text-field v-model="newItem.title" density="compact" label="Title"
                :rules="[rules.titleRequired, rules.titleUnique]" v-show="displayTitle"></v-text-field>
              <v-text-field v-model="newItem.description" density="compact" label="Description"
                :rules="[rules.descriptionRequired]"></v-text-field>
              <v-text-field v-model="newItem.deadline" density="compact" label="Deadline" type="date"
                :rules="[rules.deadlineRequired]"></v-text-field>
              <v-radio-group inline label="Priority" v-model="newItem.priority" desnsity="compact">
                <v-radio label="Low" value="low"></v-radio>
                <v-radio label="Med" value="med"></v-radio>
                <v-radio label="High" value="high"></v-radio>
              </v-radio-group>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn variant="elevated" prepend-icon="mdi-plus-circle" size="small" color="primary" @click="addItem"
                v-show="addDialog" type="submit">Add</v-btn>
              <v-btn variant="elevated" prepend-icon="mdi-square-edit-outline" size="small" color="primary"
                @click="saveItem" v-show="addDialog == false" type="submit">edit</v-btn>
              <v-btn variant="elevated" prepend-icon="mdi-cancel" size="small" color="red"
                @click="taskDialog = false">Cancel</v-btn>
            </v-card-actions>
          </v-form>
        </v-card>
      </v-dialog>
      <v-snackbar v-model="toaster" :timeout="timeout" location="bottom right">
        {{ toastertext }}
      </v-snackbar>

    </v-responsive>
  </v-container>
</template>
  
<script setup>
//
</script>
<script>
export default {
  data() {
    return {
      todolist: [],
      taskDialog: false,
      taskDialogTitle: '',
      taskDialogTitleIcon: '',
      displayTitle: true,
      addDialog: true,
      newItem: {
        title: '',
        description: '',
        deadline: new Date().toISOString().slice(0, 10),
        priority: 'low',
        iscomplete: false,
      },
      toaster: false,
      toastertext: '',
      timeout: 2000,
      rules: {
        titleRequired: value => !!value || 'Title is required',
        descriptionRequired: value => !!value || 'Description is required',
        deadlineRequired: value => !!value || 'Deadline is required',
        titleUnique: value => this.isTitleUnique(value) || 'Duplicate title',
      },
    }
  },
  methods: {
    createDialog() {
      this.resetNewItem()
      this.taskDialogTitle = 'Add Task'
      this.taskDialogTitleIcon = 'mdi-plus-circle-outline'
      this.addDialog = true
      this.displayTitle = true
      this.taskDialog = true
    },
    updateDialog(item) {
      this.taskDialogTitle = 'Edit Task'
      this.taskDialogTitleIcon = 'mdi-square-edit-outline'
      this.newItem = {
        title: item.title,
        description: item.description,
        deadline: item.deadline,
        priority: item.priority,
        iscomplete: item.iscomplete,
      }
      this.addDialog = false
      this.displayTitle = false
      this.taskDialog = true
    },
    isTitleUnique(title) {
      var foundIndex = this.todolist.findIndex(x => x.title == title)
      if (foundIndex >= 0) {
        return false
      }
      return true
    },
    addItem() {
      if (
        !this.newItem.title ||
        !this.newItem.description ||
        !this.newItem.deadline ||
        !this.newItem.priority
      ) {
        return
      }
      var foundIndex = this.todolist.findIndex(
        x => x.title == this.newItem.title
      )
      if (foundIndex >= 0) {
        return
      }
      this.todolist.push(this.newItem)
      this.taskDialog = false
      this.resetNewItem()
      this.toastertext = 'Task was added successfully!'
      this.toaster = true
    },
    saveItem() {
      if (
        !this.newItem.description ||
        !this.newItem.deadline ||
        !this.newItem.priority
      ) {
        return
      }
      var foundIndex = this.todolist.findIndex(
        x => x.title == this.newItem.title
      )
      this.todolist[foundIndex].description = this.newItem.description
      this.todolist[foundIndex].deadline = this.newItem.deadline
      this.todolist[foundIndex].priority = this.newItem.priority
      this.taskDialog = false
      this.resetNewItem()
      this.toastertext = 'Task was updated successfully!'
      this.toaster = true
    },
    deleteItem(item) {
      var foundIndex = this.todolist.findIndex(x => x.title == item.title)
      this.todolist =
        foundIndex >= 0
          ? [
            ...this.todolist.slice(0, foundIndex),
            ...this.todolist.slice(foundIndex + 1),
          ]
          : this.todolist
      this.toastertext = 'Task was deleted successfully!'
      this.toaster = true
    },
    validateNew() {
      if (
        !this.newItem.title ||
        !this.newItem.description ||
        !this.newItem.deadline ||
        !this.newItem.priority
      ) {
        return false
      }
      return true
    },
    resetNewItem() {
      this.newItem = {
        title: '',
        description: '',
        deadline: new Date().toISOString().slice(0, 10),
        priority: 'low',
        iscomplete: false,
      }
    },
    formatDate(date) {
      if (!date) return null
      const [year, month, day] = date.split('-')
      return month + '/' + day + '/' + year
    },
  },
}
</script>
