<template>
	<div class="max-w-3xl mx-auto bg-white rounded-lg mt-6">
	  <h2 class="text-2xl font-semibold text-gray-800 mb-4">Student Details</h2>
	  
	  <!-- Actions -->
	  <div class="flex justify-end mb-4">
		<Button @click="toggleEditMode" variant="outline" theme="blue">{{ editMode ? 'Cancel' : 'Edit' }}</Button>
	  </div>
  
	  <!-- Form Sections -->
	  <div class="space-y-6">
		<Section title="Basic Information">
		  <FormControl v-model="formData.full_name" type="text" label="Full Name" :disabled="!editMode" />
		  <FormControl v-model="formData.user" type="text" label="Email" :disabled="true" />
		  <FormControl v-model="formData.student_code" type="text" label="Student Code" :disabled="true" />
		</Section>
		
		<Section title="SSC Details">
		  <FormControl v-model="formData.ssc_institute_name" type="text" label="Institute Name" :disabled="!editMode" />
		  <FormControl v-model="formData.ssc_year_of_completion" type="number" label="Year of Completion" :disabled="!editMode" />
		  <FormControl v-model="formData.ssc_grade" type="number" label="Grade" :disabled="!editMode" />
		</Section>
		
		<Section title="HSC Details">
		  <FormControl v-model="formData.hsc_institute_name" type="text" label="Institute Name" :disabled="!editMode" />
		  <FormControl v-model="formData.hsc_year_of_completion" type="number" label="Year of Completion" :disabled="!editMode" />
		  <FormControl v-model="formData.hsc_grade" type="number" label="Grade" :disabled="!editMode" />
		</Section>
		
		<Section title="Undergraduate Details">
		  <FormControl v-model="formData.ug_institute_name" type="text" label="Institute Name" :disabled="!editMode" />
		  <FormControl v-model="formData.ug_degree_name" type="text" label="Degree Name" :disabled="!editMode" />
		  <FormControl v-model="formData.ug_current_year" type="number" label="Current Year" :disabled="!editMode" />
		</Section>
		
		<Section title="Masters Details">
		  <FormControl v-model="formData.masters_institute_name" type="text" label="Institute Name" :disabled="!editMode" />
		  <FormControl v-model="formData.masters_degree_name" type="text" label="Degree Name" :disabled="!editMode" />
		</Section>
		
		<Section title="PhD Details">
		  <FormControl v-model="formData.phd_institute_name" type="text" label="Institute Name" :disabled="!editMode" />
		  <FormControl v-model="formData.phd_research_topic" type="text" label="Research Topic" :disabled="!editMode" />
		</Section>
		
		<Section title="Extra Details">
		  <FormControl v-model="formData.status" type="text" label="Status" :disabled="!editMode" />
		  <FormControl v-model="formData.linkedin_profile_optional" type="text" label="LinkedIn Profile" :disabled="!editMode" />
		</Section>
	  </div>
  
	  <!-- Save Button -->
	  <div class="mt-6 text-right">
		<Button v-if="editMode" @click="updateStudentDetails" :loading="updating" variant="solid" theme="green">
		  Save Changes
		</Button>
	  </div>
	</div>
  </template>
  
  <script setup>
  import { ref, watch } from 'vue'
  import { call, Button, FormControl } from 'frappe-ui'
  import Section from '@/components/Section.vue' // A simple wrapper component for sections
  
  const studentDetails = ref(null)
  const formData = ref({})
  const loading = ref(false)
  const error = ref(null)
  const editMode = ref(false)
  const updating = ref(false)
  const props = defineProps({ profile: Object })
  
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
		formData.value = { ...studentDetails.value }
	  } else {
		studentDetails.value = null
	  }
	} catch (err) {
	  error.value = err.message || 'An error occurred'
	}
	loading.value = false
  }
  
  function toggleEditMode() {
	editMode.value = !editMode.value
	if (!editMode.value) {
	  formData.value = { ...studentDetails.value } // Reset changes if canceled
	}
  }
  
  async function updateStudentDetails() {
	updating.value = true;
	try {
		// Update the document
		await call('frappe.client.set_value', {
		doctype: 'Student Extra Details',
		name: studentDetails.value.name,
		fieldname: { ...formData.value },
		});

		// Immediately fetch the updated document
		const latestData = await call('frappe.client.get', {
		doctype: 'Student Extra Details',
		name: studentDetails.value.name,
		});

		// Update local state silently
		studentDetails.value = latestData;
		formData.value = { ...latestData };
		editMode.value = false;
	} catch (err) {
		console.error("Failed to update details:", err.message);
	}
	updating.value = false;
	}


  
  watch(() => props.profile, fetchStudentDetails, { immediate: true })
  </script>
  