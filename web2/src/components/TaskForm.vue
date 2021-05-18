<template>
  <v-form
    ref="form"
    lazy-validation
    v-model="formValid"
    v-if="item != null"
  >
    <v-alert
      :value="formError"
      color="error"
      class="pb-2"
    >{{ formError }}</v-alert>

    <v-text-field
      v-model="item.playbook"
      label="Playbook Override"
      :disabled="formSaving"
    ></v-text-field>

    <v-textarea
      v-model="item.environment"
      label="Environment Override (*MUST* be valid JSON)"
      :disabled="formSaving"
      rows="4"
    ></v-textarea>

    <v-textarea
      v-model="item.arguments"
      label="Extra CLI Arguments"
      :disabled="formSaving"
      rows="4"
    ></v-textarea>

    <v-select
      label="Select host"
      v-model="item.host"
      :items="hosts"
      item-value="id"
      item-text="host-name"
      :rules="[v => !!v || 'Host is required']"
      required
      :disabled="formSaving"
    ></v-select>
  </v-form>
</template>
<script>
import axios from 'axios';
import ItemFormBase from '@/components/ItemFormBase';

export default {
  data() {
    return {
      hosts: [],
    };
  },
  mixins: [ItemFormBase],
  props: {
    templateId: Number,
  },
  watch: {
    needReset(val) {
      if (val) {
        this.item.template_id = this.templateId;
      }
    },

    templateId(val) {
      this.item.template_id = val;
    },
  },
  async created() {
    this.item.template_id = this.templateId;
    await this.getInventoryHosts();
  },
  methods: {
    getItemsUrl() {
      return `/api/project/${this.projectId}/tasks`;
    },
    async getInventoryIdByTemplate() {
      const template = (await axios({
        method: 'get',
        url: `/api/project/${this.projectId}/templates/${this.item.template_id}`,
        responseType: 'json',
      })).data;
      return template.inventory_id;
    },
    async getInventoryHosts() {
      const inventoryId = await this.getInventoryIdByTemplate();
      this.hosts = (await axios({
        method: 'get',
        url: `/api/project/${this.projectId}/inventory/${inventoryId}/hosts`,
        responseType: 'json',
      })).data;
    },

  },
};
</script>
