<template>
    <!-- Edit Student Modal -->
    <Dialog :options="{ title: 'Edit Student Details' }" v-model:show="show">
      <template #body-content>
        <div class="space-y-3">
          <FormControl v-model="editData.full_name" type="text" label="Full Name" />
          <FormControl v-model="editData.college_name" type="text" label="College Name" />
          <FormControl v-model="editData.status" type="text" label="Status" />
          <FormControl v-model="editData.ssc_institute_name" type="text" label="SSC Institute Name" />
          <FormControl v-model="editData.ssc_year_of_completion" type="number" label="SSC Year of Completion" />
          <FormControl v-model="editData.ssc_grade" type="number" label="SSC Grade" />
          <!-- Add any other fields here as necessary -->
        </div>
      </template>
  
      <template #actions>
        <Button @click="saveChanges" variant="solid" theme="green">
          Save Changes
        </Button>
      </template>
    </Dialog>
  </template>
  
  <script setup>
  import { ref, defineProps, defineEmits } from 'vue'
  import { FormControl, Dialog, Button } from 'frappe-ui'
  
  const props = defineProps({
    studentDetails: Object, 
    show: Boolean,
  })
  
  const emit = defineEmits(['update-student', 'update:show'])
  
  const editData = ref({ ...props.studentDetails })
  
  function saveChanges() {
    emit('update-student', editData.value)
    emit('update:show', false)
  }
  </script>
  