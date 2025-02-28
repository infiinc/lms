<template>
	<div class="max-w-3xl mx-auto bg-white rounded-lg mt-6">
	  <h2 class="text-2xl font-semibold text-gray-800 mb-4">Student Details</h2>
  
	  <!-- Actions -->
	  <div class="flex justify-end mb-4">
		<Button v-if="studentDetails" @click="toggleEditMode" variant="outline" theme="blue">
		  {{ editMode ? 'Cancel' : 'Edit' }}
		</Button>
		<Button v-if="!studentDetails" @click="showCollegePrompt" variant="solid" theme="green">
		  Create Entry
		</Button>
	  </div>
  
	  <!-- College Name Selection Modal -->
	  <div v-if="showCollegeNamePrompt" class="p-4 bg-gray-100 rounded-lg">
		<h3 class="text-lg font-semibold">Select Your College</h3>
		<select v-model="selectedCollege" class="border p-2 w-full rounded">
		  <option v-for="college in collegeList" :key="college.name" :value="college.name">
			{{ college.college_name }}
		  </option>
		</select>
		<div class="mt-4 text-right">
		  <Button @click="createStudentDetailsEntry" :disabled="!selectedCollege" variant="solid" theme="blue">
			Submit
		  </Button>
		</div>
	  </div>
  
	  <!-- Form Sections -->
	  <div class="space-y-6" v-if="studentDetails">
		<Section title="Basic Information">
		  <FormControl v-model="formData.full_name" type="text" label="Full Name" :disabled="!editMode" />
		  <FormControl v-model="formData.user" type="text" label="Email" :disabled="true" />
		  <FormControl v-model="formData.student_code" type="text" label="Student Code" :disabled="true" />
		</Section>
  
		<Section title="Undergraduate Details">
		  <FormControl v-model="formData.ug_institute_name" type="text" label="College Name" :disabled="true" />
		  <FormControl v-model="formData.ug_degree_name" type="text" label="Degree Name" :disabled="!editMode" />
		  <FormControl v-model="formData.ug_current_year" type="number" label="Current Year" :disabled="!editMode" />
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
  import { ref, watch, onMounted } from 'vue';
  import { call, Button, FormControl } from 'frappe-ui';
  import Section from '@/components/Section.vue';
  
  const studentDetails = ref(null);
  const formData = ref({});
  const loading = ref(false);
  const error = ref(null);
  const editMode = ref(false);
  const updating = ref(false);
  const showCollegeNamePrompt = ref(false);
  const selectedCollege = ref('');
  const collegeList = ref([]);
  const props = defineProps({ profile: Object });
  
  async function fetchStudentDetails() {
	loading.value = true;
	error.value = null;
	try {
	  let result = await call('frappe.client.get_list', {
		doctype: 'Student Extra Details',
		filters: { user: props.profile.data.name },
		fields: ['name'],
		limit_page_length: 1,
	  });
  
	  if (result.length > 0) {
		studentDetails.value = await call('frappe.client.get', {
		  doctype: 'Student Extra Details',
		  name: result[0].name,
		});
		formData.value = { ...studentDetails.value };
	  } else {
		studentDetails.value = null;
	  }
	} catch (err) {
	  error.value = err.message || 'An error occurred';
	}
	loading.value = false;
  }
  
  function toggleEditMode() {
	editMode.value = !editMode.value;
	if (!editMode.value) {
	  formData.value = { ...studentDetails.value };
	}
  }
  
  async function updateStudentDetails() {
	updating.value = true;
	try {
	  await call('frappe.client.set_value', {
		doctype: 'Student Extra Details',
		name: studentDetails.value.name,
		fieldname: { ...formData.value },
	  });
  
	  const latestData = await call('frappe.client.get', {
		doctype: 'Student Extra Details',
		name: studentDetails.value.name,
	  });
  
	  studentDetails.value = latestData;
	  formData.value = { ...latestData };
	  editMode.value = false;
	} catch (err) {
	  console.error("Failed to update details:", err.message);
	}
	updating.value = false;
  }
  
  async function fetchCollegeList() {
	try {
	  let colleges = await call('frappe.client.get_list', {
		doctype: 'College',
		fields: ['name', 'college_name']
	  });
	  collegeList.value = colleges;
	} catch (err) {
	  console.error("Failed to fetch college list:", err.message);
	}
  }
  
  function showCollegePrompt() {
	showCollegeNamePrompt.value = true;
	fetchCollegeList();
  }
  
  async function createStudentDetailsEntry() {
	if (!selectedCollege.value) return;
  
	try {
	  const newEntry = await call('frappe.client.insert', {
		doc: {
		  doctype: 'Student Extra Details',
		  user: props.profile.data.name,
		  current_college: selectedCollege.value
		}
	  });
  
	  studentDetails.value = newEntry;
	  formData.value = { ...newEntry };
	  showCollegeNamePrompt.value = false;
	} catch (err) {
	  console.error("Failed to create Student Extra Details entry:", err.message);
	}
  }
  
  watch(() => props.profile, fetchStudentDetails, { immediate: true });
  
  onMounted(fetchCollegeList);
  </script>
  