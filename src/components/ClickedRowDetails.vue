<template>
  <v-row>
    <v-col cols="12">
      <div class="d-flex justify-center">
        <v-card color="blue" width="800">
          <v-card-title> ID: {{ clickedRow.id }}</v-card-title>
          <v-card-title>UserID: {{ clickedRow.userId }}</v-card-title>
          <v-card-title class="text-h6 text-md-h6 text-lg-h4">
            {{clickedRow.title}}
          </v-card-title>
          <v-card-text>
            {{ clickedRow.body }}
          </v-card-text>
        </v-card>
      </div>
    </v-col>
    <v-col class="d-flex justify-center">
      <v-btn color="grey" @click="$emit('goBack')">Go Back</v-btn>
    </v-col>
  </v-row>
</template>

<script>
export default {
  props: {
    clickedRow: {
      type: Object,
      required: true,
    },
  },
};
</script>
