<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" :rules="nameValidationRules" required />
        <q-input v-model="tempData.age" label="年齡" :rules="ageValidationRules" required type="number" />
        <q-btn color="primary" class="q-mt-md" @click="handleUpdate">更新</q-btn>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps, useQuasar } from 'quasar';
import { ref } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
const blockData = ref([
  {
    name: 'test',
    age: 25,
  },
]);
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

const tempData = ref({
  name: '',
  age: '',
});

const $q = useQuasar();

const nameValidationRules = [
  (val) => !!val || '姓名不得為空',
];

const ageValidationRules = [
  (val) => !!val || '年齡不得為空',
  (val) => /^\d+$/.test(val) || '年齡請輸入正整數',
];

function handleClickOption(btn, data) {
  const fetchData = async () => {
    try {
      const response = await axios.get('https://demo.mercuryfire.com.tw:49110/crudTest/a');
      blockData.value = response.data;
    } catch (error) {
      console.error('Error fetching data:', error);
    }
  };

  const handleSubmit = async () => {
    try {
      const response = await axios.post('https://demo.mercuryfire.com.tw:49110/crudTest', tempData.value);
      console.log('Data added successfully:', response.data);
      fetchData();
    } catch (error) {
      console.error('Error adding data:', error);
    }
  };

  const handleClickOption = async (btn, data) => {
    switch (btn.status) {
      case 'edit':

        break;
      case 'delete':

        try {
          await axios.delete(`https://demo.mercuryfire.com.tw:49110/crudTest/${data.id}`);
          console.log('Data deleted successfully');
          fetchData();
        } catch (error) {
          console.error('Error deleting data:', error);
        }
        break;
      default:
        break;
    }
  };

  const handleUpdate = async () => {
    if (!currentId) {
      console.error('No data selected for update');
      return;
    }

    try {
      const response = await axios.patch(`https://demo.mercuryfire.com.tw:49110/crudTest/${currentId}`, tempData.value);
      console.log('Data updated successfully:', response.data);
      fetchData();
    } catch (error) {
      console.error('Error updating data:', error);
    }
  };

  fetchData();

  return {
    blockData,
    tableConfig,
    tableButtons,
    tempData,
    handleSubmit,
    handleClickOption,
  };
}
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
