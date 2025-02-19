<template>
	<div class="max-w-2xl mx-auto bg-white shadow-md rounded-lg p-6 mt-6">
	  <h2 class="text-2xl font-semibold text-gray-800 mb-4">Student Details</h2>
  
	  <!-- Reload Button -->
	  <div class="flex justify-between items-center">
		<Button @click="fetchStudentDetails()" :loading="loading" variant="solid" theme="blue">
		  Reload
		</Button>
		<Button @click="openEditModal()" variant="outline" theme="orange">Edit</Button>
	  </div>
  
	  <!-- Loading State -->
	  <div v-if="loading" class="mt-4 text-gray-500">Fetching data...</div>
  
	  <!-- Error State -->
	  <div v-else-if="error" class="mt-4 text-red-500">Error: {{ error }}</div>
  
	  <!-- Student Details -->
	  <div v-else-if="studentDetails" class="mt-4 space-y-4">
		<div><strong class="text-gray-700">Full Name:</strong> <span class="text-gray-900">{{ studentDetails.full_name }}</span></div>
		<div><strong class="text-gray-700">Email:</strong> <span class="text-gray-900">{{ studentDetails.user }}</span></div>
		<div><strong class="text-gray-700">Student Code:</strong> <span class="text-gray-900">{{ studentDetails.student_code }}</span></div>
		<div><strong class="text-gray-700">College Name:</strong> <span class="text-gray-900">{{ studentDetails.college_name || 'N/A' }}</span></div>
		<div><strong class="text-gray-700">SSC Institute:</strong> <span class="text-gray-900">{{ studentDetails.ssc_institute_name || 'N/A' }}</span></div>
		<div><strong class="text-gray-700">SSC Year of Completion:</strong> <span class="text-gray-900">{{ studentDetails.ssc_year_of_completion || 'N/A' }}</span></div>
		<div><strong class="text-gray-700">SSC Grade:</strong> <span class="text-gray-900">{{ studentDetails.ssc_grade || 'N/A' }}</span></div>
		<div><strong class="text-gray-700">Status:</strong> <span class="text-gray-900">{{ studentDetails.status }}</span></div>
		<div><strong class="text-gray-700">HSC Institute:</strong> <span class="text-gray-900">{{ studentDetails.hsc_institute_name || 'N/A' }}</span></div>
		<div><strong class="text-gray-700">UG Institute:</strong> <span class="text-gray-900">{{ studentDetails.ug_institute_name || 'N/A' }}</span></div>
		<div><strong class="text-gray-700">Masters Institute:</strong> <span class="text-gray-900">{{ studentDetails.masters_institute_name || 'N/A' }}</span></div>
		<div><strong class="text-gray-700">PhD Institute:</strong> <span class="text-gray-900">{{ studentDetails.phd_institute_name || 'N/A' }}</span></div>
	  </div>
  
	  <!-- No Data -->
	  <div v-else class="text-gray-500 mt-4">No student details found.</div>
  
	  <!-- Edit Modal -->
	  <Dialog :options="{ title: 'Edit Student Details' }" v-model="showEditModal">
		<template #body-content>
		  <div class="space-y-3">
			<FormControl v-model="editData.full_name" type="text" label="Full Name" />
			<FormControl v-model="editData.college_name" type="text" label="College Name" />
			<FormControl v-model="editData.ssc_institute_name" type="text" label="SSC Institute Name" />
			<FormControl v-model="editData.ssc_year_of_completion" type="number" label="SSC Year of Completion" />
			<FormControl v-model="editData.ssc_grade" type="number" label="SSC Grade" />
			<FormControl v-model="editData.status" type="text" label="Status" />
			<FormControl v-model="editData.hsc_institute_name" type="text" label="HSC Institute Name" />
			<FormControl v-model="editData.ug_institute_name" type="text" label="UG Institute Name" />
			<FormControl v-model="editData.masters_institute_name" type="text" label="Masters Institute Name" />
			<FormControl v-model="editData.phd_institute_name" type="text" label="PhD Institute Name" />
		  </div>
		</template>
  
		<template #actions>
		  <Button @click="updateStudentDetails" :loading="updating" variant="solid" theme="green">
			Save Changes
		  </Button>
		</template>
	  </Dialog>
	</div>
  </template>
  
  <script setup>
  import { ref } from 'vue'
  import { call, Badge, Button, Dialog, FormControl } from 'frappe-ui'
  
  const studentDetails = ref(null)
  const loading = ref(false)
  const error = ref(null)
  
  const showEditModal = ref(false)
  const editData = ref({})
  const updating = ref(false)

  const props = defineProps({
	profile: {
		type: Object,
		required: true,
	},
})
  
  async function fetchStudentDetails() {
	loading.value = true
	error.value = null
  
	try {
	  let result = await call('frappe.client.get_list', {
		doctype: 'Student Extra Details',
		filters: { user: props.profile.data.name },
		fields: ['name'],
		limit_page_length: 1,
	  })
  
	  if (result.length > 0) {
		studentDetails.value = await call('frappe.client.get', {
		  doctype: 'Student Extra Details',
		  name: result[0].name,
		})
	  } else {
		studentDetails.value = null
	  }
	} catch (err) {
	  error.value = err.message || 'An error occurred'
	}
  
	loading.value = false
  }
  
  // Open Edit Modal & Prefill Data
  function openEditModal() {
	if (studentDetails.value) {
	  editData.value = { ...studentDetails.value } // Clone data for editing
	  showEditModal.value = true
	}
  }
  
  // Update Student Details
  async function updateStudentDetails() {
	updating.value = true
  
	try {
	  await call('frappe.client.set_value', {
		doctype: 'Student Extra Details',
		name: studentDetails.value.name,
		fieldname: {
		  full_name: editData.value.full_name,
		  college_name: editData.value.college_name,
		  ssc_institute_name: editData.value.ssc_institute_name,
		  ssc_year_of_completion: editData.value.ssc_year_of_completion,
		  ssc_grade: editData.value.ssc_grade,
		  status: editData.value.status,
		  hsc_institute_name: editData.value.hsc_institute_name,
		  ug_institute_name: editData.value.ug_institute_name,
		  masters_institute_name: editData.value.masters_institute_name,
		  phd_institute_name: editData.value.phd_institute_name,
		},
	  })
  
	  // Update UI
	  studentDetails.value = { ...editData.value }
	  showEditModal.value = false
	} catch (err) {
	  alert('Failed to update details: ' + err.message)
	}
  
	updating.value = false
  }
  </script>
  