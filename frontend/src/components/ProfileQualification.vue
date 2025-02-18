<template>
	<div>
		<h2 class="text-xl font-semibold mb-4">Qualifications</h2>

		<div v-if="qualification.data">
			<!-- SSC Qualification -->
			<div class="mb-4">
				<h3 class="text-lg font-medium">Secondary School (SSC)</h3>
				<p><strong>Institute:</strong> {{ qualification.data.ssc_institute_name || "N/A" }}</p>
				<p><strong>Year:</strong> {{ qualification.data.ssc_year_of_completion || "N/A" }}</p>
				<p><strong>Grade:</strong> {{ qualification.data.ssc_grade || "N/A" }}</p>
				<p v-if="qualification.data.ssc_marksheet">
					<a :href="qualification.data.ssc_marksheet" target="_blank" class="text-blue-500">View Marksheet</a>
				</p>
			</div>

			<!-- HSC Qualification -->
			<div class="mb-4">
				<h3 class="text-lg font-medium">Higher Secondary (HSC)</h3>
				<p><strong>Institute:</strong> {{ qualification.data.hsc_institute_name || "N/A" }}</p>
				<p><strong>Year:</strong> {{ qualification.data.hsc_year_of_completion || "N/A" }}</p>
				<p><strong>Grade:</strong> {{ qualification.data.gradepercentage || "N/A" }}</p>
				<p v-if="qualification.data.hsc_marksheet">
					<a :href="qualification.data.hsc_marksheet" target="_blank" class="text-blue-500">View Marksheet</a>
				</p>
			</div>

			<!-- Undergraduate Qualification -->
			<div class="mb-4">
				<h3 class="text-lg font-medium">Undergraduate Degree</h3>
				<p><strong>Institute:</strong> {{ qualification.data.ug_institute_name || "N/A" }}</p>
				<p><strong>Degree:</strong> {{ qualification.data.ug_degree_name || "N/A" }}</p>
				<p><strong>Year:</strong> {{ qualification.data.degree_year_of_completion || "N/A" }}</p>
				<p><strong>Grade/CGPA:</strong> {{ qualification.data.gradecgpa || "N/A" }}</p>
				<p v-if="qualification.data.undergraduate_certificate">
					<a :href="qualification.data.undergraduate_certificate" target="_blank" class="text-blue-500">View Certificate</a>
				</p>
			</div>

			<!-- Masters Qualification -->
			<div class="mb-4" v-if="qualification.data.masters_institute_name">
				<h3 class="text-lg font-medium">Masters Degree</h3>
				<p><strong>Institute:</strong> {{ qualification.data.masters_institute_name || "N/A" }}</p>
				<p><strong>Degree:</strong> {{ qualification.data.masters_degree_name || "N/A" }}</p>
				<p><strong>Year:</strong> {{ qualification.data.masters_year_of_completion || "N/A" }}</p>
				<p><strong>Grade/CGPA:</strong> {{ qualification.data.gradecgpa_copy || "N/A" }}</p>
				<p v-if="qualification.data.masters_degree_certificate">
					<a :href="qualification.data.masters_degree_certificate" target="_blank" class="text-blue-500">View Certificate</a>
				</p>
			</div>

			<!-- PhD Qualification -->
			<div class="mb-4" v-if="qualification.data.phd_institute_name">
				<h3 class="text-lg font-medium">PhD</h3>
				<p><strong>Institute:</strong> {{ qualification.data.phd_institute_name || "N/A" }}</p>
				<p><strong>Research Topic:</strong> {{ qualification.data.research_topic || "N/A" }}</p>
				<p><strong>Year:</strong> {{ qualification.data.phd_year_of_completion || "N/A" }}</p>
				<p v-if="qualification.data.thesus_upload">
					<a :href="qualification.data.thesus_upload" target="_blank" class="text-blue-500">View Thesis</a>
				</p>
			</div>

		</div>

		<!-- Show message if no qualification details exist -->
		<div v-else class="text-gray-500">
			<p>No qualifications available.</p>
		</div>
	</div>
</template>

<script setup>
import { createResource } from 'frappe-ui'
import { defineProps } from 'vue'

const props = defineProps({
	profile: {
		type: Object,
		required: true,
	},
})

// Fetch Qualification Details for the Logged-in User
const qualification = createResource({
	url: 'frappe.client.get',
	makeParams() {
		return {
			doctype: 'Student Extra Details',
			filters: {
				user: props.profile.data?.name, // Fetch qualifications for this user
			},
		}
	},
})

watchEffect(() => {
	if (qualification.data) {
		console.log('Qualification Data:', qualification.data)
	}
})
</script>
