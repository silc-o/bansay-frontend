<template>
  <div class="q-pa-md">

    <h5>Pending Approvals</h5>

    <q-table
      :rows="pendingUsers"
      :columns="columns"
      row-key="id"
      flat
      bordered
      :loading="loading"
    >
      <template #body-cell-actions="props">
        <q-td>
          <q-btn
            color="primary"
            label="Approve"
            size="sm"
            @click="approveUser(props.row.id)"
          />
        </q-td>
      </template>
    </q-table>

    <q-banner
      v-if="error"
      class="bg-negative text-white q-mt-md"
    >
      {{ error }}
    </q-banner>

  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { BansayService } from 'src/services/bansay-service';
const pendingUsers = ref<any[]>([]);
const loading = ref(false);
const error = ref('');

const columns = [
  { name: 'username', label: 'Username', field: 'username', align: 'left' },
  { name: 'email', label: 'Email', field: 'email', align: 'left' },
  { name: 'role', label: 'Role', field: 'role', align: 'center' },
  { name: 'status', label: 'Status', field: 'status', align: 'center' },
];


const loadPendingUsers = async () => {
  loading.value = true;
  error.value = '';

  try {
    const api = BansayService.getInstance();
    const response = await api.getUsers({ status: 'pending' });

    pendingUsers.value = response.data;
  } catch (err: any) {
    console.error(err);
    error.value = 'Failed to load pending users.';
  } finally {
    loading.value = false;
  }
};

const approveUser = async (id: number) => {
  try {
    const api = BansayService.getInstance();

    await api.patchUser(id, { status: 'active' });

    // Remove from the list after approving
    pendingUsers.value = pendingUsers.value.filter(u => u.id !== id);
  } catch (err: any) {
    console.error(err);
    error.value = 'Failed to approve user.';
  }
};

onMounted(() => {
  loadPendingUsers();
});
</script>
