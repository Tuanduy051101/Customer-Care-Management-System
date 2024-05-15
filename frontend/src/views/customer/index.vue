<script>
import Table from "../../components/table/table_customer.vue";
import Add from "./add.vue";
import Edit from "./edit.vue";
import View from "./view.vue";
import AddHabit from "./addHabit.vue";
import { reactive, computed, ref, onBeforeMount, watch } from "vue";
import {
  http_getAll,
  http_deleteOne,
  alert_success,
  alert_error,
  alert_warning,
  alert_delete,
  alert_delete_wide,
  Pagination,
  Dropdown,
  Select,
  Search,
  Select_Advanced,
  DeleteAll,
  Company_KH,
  Customer_Work,
  Customer,
  Customer_Types,
  Status_Task,
  formatDate,
  formatDateTime,
  formatDateTime_2,
} from "../common/import";

import {
  isDeleteCustomer,
  isEditCustomer,
  isReadCustomer,
  isCreateCustomer,
  isCreateHabit,
} from "../../use/getSessionItem";

import { useRouter } from "vue-router";

export default {
  components: {
    Table,
    Pagination,
    Dropdown,
    Select,
    Search,
    Add,
    DeleteAll,
    Edit,
    View,
    AddHabit,
    Select_Advanced,
  },
  setup(ctx) {
    const data = reactive({
      activeShowEdit: false,
      activeShowAdd: false,
      resetDataAdd: false,
      Events: [],
      items: [],
      entryValue: 5,
      numberOfPages: 1,
      totalRow: 0,
      startRow: 0,
      endRow: 0,
      currentPage: 1,
      searchText:  sessionStorage.getItem("searchCustomer") || '',
      itemAdd: {
        name: "",
        birthdate: "",
        address: "",
        phone: "",
        email: "",
        type: "",
        avatar: "",
        wor_current_workplace: "",
        wor_work_history: "",
        wor_current_position: "",
        wor_work_temp: "",
        companyKH: "",
      },
      activeEdit: false,
      viewValue: {
        Customer: {
          name: "",
          birthday: "",
          avatar: "",
          phone: "",
          email: "",
          address: "",
        },
        Customer_Type: {
          name: "",
        },
        Company_KH: {
          name: "",
        },
        Events: [
          {
            name: "",
            content: "",
            time_duration: "",
          },
        ],
        Habits: [
          {
            name: "",
          },
        ],
        current_workplace: "",
        work_history: "",
        current_position: "",
        work_temp: "",
      },

      customerTypeValue: "",
      customerValue: {},
      showAddHabit: false,
      customerType: null,
      customerStatus: null, // khởi tạo trạng thái chăm sóc khách hàng
      choseSearch: "",
      selectAll: [
        {
          checked: false,
        },
      ],
      viewCareCus: null,
      habitAdd: [
        {
          name: "",
        },
      ],
      eventAdd: {
        name: "",
        content: "",
        time_duration: "eventAdd.start_time",
        start_time: "",
        end_time: "",
        place: "",
      },
    });

    const entryValueCustomerType = ref("");
    const entryNameCustomerType = ref("Loại khách hàng");
    const entryValueStatusTask = ref("");
    const entryNameStatusTask = ref("Trạng thái chăm sóc");

    const reFresh = async () => {
      const cusWork = await http_getAll(Customer_Work);
      const customerType = await http_getAll(Customer_Types);
      const customerStatus = await http_getAll(Status_Task);

      data.items = cusWork.documents;
      for (let value of data.items) {
        value.checked = false;
      }

      // loại khách hàng
      data.customerType = customerType.documents;

      // format loại khách hàng
      data.customerType = data.customerType.map((value, index) => {
        return {
          ...value,
          value: value._id,
        };
      });

      // trạng thái task chăm sóc khách hàng
      data.customerStatus = customerStatus.map((value, index) => {
        return {
          ...value,
          value: value._id,
        };
      });

      if (entryValueCustomerType.value.length > 0) {
        data.items = data.items.filter((cusWork) => {
          return (
            cusWork.Customer.Customer_Type._id == entryValueCustomerType.value
          );
        });
      }

      if (entryValueStatusTask.value.length > 0) {
        data.items = data.items.filter((cusWork) => {
          return (
            cusWork.Customer.Tasks.filter((task) => {
              return task.Status_Task._id == entryValueStatusTask.value;
            }).length > 0
          );
        });
      }
    };

    const reFresh1 = async () => {
      const cusWork = await http_getAll(Customer_Work);
      const customerType = await http_getAll(Customer_Types);
      const customerStatus = await http_getAll(Status_Task);

      const temp = cusWork.documents;
      // data.items = [...new Set([...temp, ...data.items])];
      for (let i = 0; i < data.items.length; i++) {
        temp[i].checked = data.items[i].checked;
      }
      data.items = temp;
    };

    const showAddHabit = () => {
      data.customerValue = {};
      data.showAddHabit = false;
      for (let value of data.items) {
        if (value.checked == true) {
          data.customerValue = value;
          data.showAddHabit = true;
          break;
        }
      }
      if (data.showAddHabit == false) {
        alert_warning(`Thêm thói quen khách hàng`, `Vui lòng chọn khách hàng.`);
      }
    };
    const router = useRouter();
    onBeforeMount(() => {
      reFresh();
    });

    // computed
    const toString = computed(() => {
      if (data.choseSearch == "name") {
        return data.items.map((value, index) => {
          return [value.Customer.name].join("").toLocaleLowerCase();
        });
      } else if (data.choseSearch == "email") {
        return data.items.map((value, index) => {
          return [value.Customer.email].join("").toLocaleLowerCase();
        });
      } else if (data.choseSearch == "phone") {
        return data.items.map((value, index) => {
          return [value.Customer.phone].join("").toLocaleLowerCase();
        });
      } else {
        return data.items.map((value, index) => {
          return [
            value.Customer.name,
            value.Customer.email,
            value.Customer.phone,
          ]
            .join("")
            .toLocaleLowerCase();
        });
      }
    });
    const filter = computed(() => {
      return data.items.filter((value, index) => {
        return toString.value[index].includes(
          data.searchText.toLocaleLowerCase()
        );
      });
    });
    const filtered = computed(() => {
      if (!data.searchText) {
        data.totalRow = data.items.length;
        return data.items;
      } else {
        data.totalRow = filter.value.length;
        return filter.value;
      }
    });
    const setNumberOfPages = computed(() => {
      return Math.ceil(filtered.value.length / data.entryValue);
    });
    const setPages = computed(() => {
      if (data.items.length > 0) {
        if (setNumberOfPages.value == 0 || data.entryValue == "All") {
          data.entryValue = data.items.length;
          data.numberOfPages = 1;
        } else data.numberOfPages = setNumberOfPages.value;
        data.startRow = (data.currentPage - 1) * data.entryValue + 1;
        data.endRow = data.currentPage * data.entryValue;
        return filtered.value.filter((item, index) => {
          return (
            index + 1 > (data.currentPage - 1) * data.entryValue &&
            index + 1 <= data.currentPage * data.entryValue
          );
        });
      } else return data.items.value;
    });
    // methods
    const update = (item) => {
      // console.log("updating", item);
    };
    const deleteOne = (_id) => {
      // console.log("deleting", _id);
    };

    // watch
    const activeMenu = ref(1);

    // CRUD

    const handleDelete = async (customerId, CompanyId, item) => {
      const isConfirmed = await alert_delete(
        "Xóa",
        `Bạn có chắc là xóa khách hàng ${item.Customer.name} không!!`
      );

      if (isConfirmed) {
        const rsCustomer = await http_deleteOne(Customer, customerId);
        if (rsCustomer.error) {
          alert_error("Lổi ", rsCustomer.msg);
        } else {
          reFresh();
          alert_success("Thành công", "Xóa khách hàng thành công");
        }
      }
    };

    const refresh_customer = () => {
      data.resetDataAdd = false;
      reFresh();
    };

    const view = (item) => {
      data.viewValue = {
        Customer: {
          _id: item.Customer._id,
          name: item.Customer.name,
          birthday: formatDate(item.Customer.birthday),
          avatar: item.Customer.avatar,
          phone: item.Customer.phone,
          email: item.Customer.email,
          address: item.Customer.address,
        },
        Customer_Type: {
          _id: item.Customer.Customer_Type._id,
          name: item.Customer.Customer_Type.name,
        },
        Company_KH: {
          _id: item.Company_KH._id,
          name: item.Company_KH.name,
        },
        Events: [...item.Customer.Events],
        Tasks: [...item.Customer.Tasks],
        Habits: [...item.Customer.Habits],
        _id: item._id,
        current_workplace: item.current_workplace,
        work_history: item.work_history,
        current_position: item.current_position,
        work_temp: item.work_temp,
      };

      data.viewCareCus = item.Customer.Tasks.map((value) => {
        return {
          start_date: formatDate(value.start_date),
          end_date: formatDate(value.end_date),
          content: value.content,
          customerName: item.Customer.name,
          cycleName: value.Cycle.name, // join bản sao
          statusName: value.Status_Task.name,
          EvaluateStar: value.Evaluate.star,
          comment:
            value.Comment == null ? "Chưa cập nhật" : value.Comment.content,
        };
      });

      data.Events = item.Customer.Events.map((item) => {
        return {
          // 'name','time_duration', 'content'
          name: item.name,
          time_duration: formatDateTime_2(item.time_duration),
          content: item.content,
          place: item.place != null ? item.place : "không có",
        };
      });
    };

    //   formatDateTime,
    // formatDate,
    const edit = (item, isCheck) => {
      data.activeShowEdit = true;
      reFresh();

      data.viewValue = {
        Customer: {
          _id: item.Customer._id,
          name: item.Customer.name,
          birthday: item.Customer.birthday,
          avatar: item.Customer.avatar,
          phone: item.Customer.phone,
          email: item.Customer.email,
          address: item.Customer.address,
        },
        Customer_Type: {
          _id: item.Customer.Customer_Type._id,
          name: item.Customer.Customer_Type.name,
        },
        Company_KH: {
          _id: item.Company_KH._id,
          name: item.Company_KH.name,
        },
        _id: item._id,
        current_workplace: item.current_workplace,
        work_history: item.work_history,
        current_position: item.current_position,
        work_temp: item.work_temp,
      };

      data.activeEdit = isCheck;
    };

    const updateEntryValueCustomerType = (value) => {
      entryValueCustomerType.value = value;
    };

    const updateEntryValueStatusTask = (value) => {
      entryValueStatusTask.value = value;
    };

    const handleSelectAll = (value) => {
      if (value == false) {
        for (let value1 of data.items) {
          value1.checked = true;
        }
      } else {
        for (let value1 of data.items) {
          value1.checked = false;
        }
      }
    };

    const deleteMany = async () => {
      try {
        const deleteArray = data.items.filter((value, index) => {
          return value.checked == true;
        });
        let name, phone, email;
        let contentAlert = `<p>Bạn có muốn xoá tất cả khách hàng này không?</p><p>Tổng số khách hàng sẽ xoá là: <span style="color: blue;">${deleteArray.length}</span></p>
          <table class="table table-bordered">
      <thead>
        <tr>
          <th>Tên khách hàng</th>
          <th>Email</th>
          <th>Số điện thoại</th>
        </tr>
      </thead> <tbody>`;

        for (let value of deleteArray) {
          contentAlert += `<tr>
          <td>${value.Customer.name}</td>
          <td>${value.Customer.email}</td>
          <td>
            ${value.Customer.phone}
          </td>
        </tr>`;
        }
        contentAlert += `</tbody>
    </table>`;
        const isConfirmed = await alert_delete_wide(
          `Xoá nhiều khách khách hàng`,
          contentAlert
        );
        if (isConfirmed) {
          let checkDeleteAll = false;
          for (let valueDelete of deleteArray) {
            const rsCustomer = await http_deleteOne(
              Customer,
              valueDelete.Customer._id
            );
            if (rsCustomer.error) {
              alert_error("Lổi ", rsCustomer.msg);
              checkDeleteAll = false;
            } else {
              checkDeleteAll = true;
            }
          }
          if (checkDeleteAll) {
            reFresh();
            alert_success("Thành công", "Xóa khách hàng thành công");
          }
        }
      } catch (error) {
        console.log(error);
      }
    };

    watch(entryValueCustomerType, (newValue, oldValue) => {
      if (newValue != "") {
        data.currentPage = 1;
        reFresh();
      } else {
        data.currentPage = 1;
        reFresh();
      }
    });

    watch(entryValueStatusTask, (newValue, oldValue) => {
      if (newValue != "") {
        data.currentPage = 1;
        reFresh();
      } else {
        data.currentPage = 1;
        reFresh();
      }
    });

    const removeItem = (index) => {
      data.habitAdd = data.habitAdd.filter((value1, index1) => {
        return index1 != index;
      });
    };
    console.log(sessionStorage.getItem("searchCustomer"))
    return {
      update,
      deleteOne,
      edit,
      handleDelete,
      refresh_customer,
      view,
      showAddHabit,
      updateEntryValueCustomerType,
      updateEntryValueStatusTask,
      entryValueCustomerType,
      entryNameCustomerType,
      entryValueStatusTask,
      entryNameStatusTask,
      data,
      setPages,
      activeMenu,
      handleSelectAll,
      deleteMany,
      reFresh,
      removeItem,
      reFresh1,
      isDeleteCustomer,
      isEditCustomer,
      isReadCustomer,
      isCreateCustomer,
      isCreateHabit,
    };
  },
};
</script>

<template>
  <div class="border-box d-flex flex-column ml-2" style="margin-right: -3px">
    <!-- Menu -->
    <div class="d-flex menu my-3 mx-3 justify-content-end">
      <router-link
        to="/customer"
        @click="activeMenu = 1"
        :class="[activeMenu == 1 ? 'active-menu' : 'none-active-menu']"
        ><span class="size-18">Khách hàng</span>
      </router-link>
      <router-link
        to="/customer_types"
        @click="activeMenu = 2"
        :class="[activeMenu == 2 ? 'active-menu' : 'none-active-menu']"
        ><span class="size-18">Loại khách hàng</span>
      </router-link>
    </div>
    <!-- Filter -->
    <div class="border-hr mb-3"></div>
    <div class="d-flex flex-column">
      <span class="mx-3 mb-3 h6 size-18">Lọc khách hàng</span>
      <div class="d-flex mx-3">
        <div class="form-group w-100">
          <!-- entryValue = modelValue -->
          <Select
            :title="`Loại khách hàng`"
            :entryValue="entryNameCustomerType"
            :options="data.customerType"
            @update:entryValue="
              (value, value1) => (
                updateEntryValueCustomerType(value),
                (entryNameCustomerType = value1.name)
              )
            "
            @refresh="
              (entryNameCustomerType = 'Loại khách hàng'),
                updateEntryValueCustomerType(''),
                (data.currentPage = 1)
            "
            style="height: 35px"
          />
        </div>
        <div class="form-group w-100 ml-3">
          <Select
            :title="`Trạng thái chăm sóc`"
            :entryValue="entryNameStatusTask"
            :options="data.customerStatus"
            @update:entryValue="
              (value, value1) => (
                updateEntryValueStatusTask(value),
                (entryNameStatusTask = value1.name)
              )
            "
            @refresh="
              (entryNameStatusTask = 'Trạng thái chăm sóc'),
                updateEntryValueStatusTask(''),
                (data.currentPage = 1)
            "
            style="height: 35px"
          />
        </div>
        <div class="form-group"></div>
      </div>
    </div>
    <!-- Search -->
    <div class="border-hr mb-3"></div>
    <div class="d-flex justify-content-between mx-3 mb-3">
      <div class="d-flex justify-content-start">
        <Select
          class="d-flex justify-content-start"
          :options="[
            {
              name: 5,
              value: 5,
            },
            {
              name: 10,
              value: 10,
            },
            {
              name: 20,
              value: 20,
            },
            {
              name: 30,
              value: 30,
            },
          ]"
          style="width: 125px"
          :title="`Số bản ghi`"
          @update:entryValue="(value) => (data.entryValue = value)"
          :entryValue="data.entryValue"
          @refresh="(data.entryValue = 'All'), (data.currentPage = 1)"
        />
        <Search
          class="ml-3"
          style="width: 300px"
          @update:searchText="(value) => (data.searchText = value)"
          :entryValue="data.searchText"
          @choseSearch="
            async (value) => (
              (data.choseSearch = value), (data.currentPage = 1)
            )
          "
          @refresh="(data.entryValue = 'All'), (data.currentPage = 1)"
          :options="[
            {
              _id: 'name',
              name: 'Tìm kiếm theo tên',
            },
            {
              _id: 'email',
              name: 'Tìm kiếm theo email',
            },
            {
              _id: 'phone',
              name: 'Tìm kiếm theo số điện thoại',
            },
          ]"
        />
      </div>
      <div class="d-flex align-items-start">
        <button
          type="button"
          class="btn btn-danger mr-3"
          data-toggle="modal"
          data-target="#model-delete-all"
          @click="deleteMany()"
          :disabled="isDeleteCustomer() ? false : true"
        >
          <span id="delete-all" class="mx-2"
            ><span class="size-16">Xoá</span></span
          >
        </button>
        <!-- <DeleteAll :items="data.items" /> -->
        <button
          type="button"
          class="btn btn-primary"
          data-toggle="modal"
          data-target="#model-add"
          @click="(data.resetDataAdd = true), (data.activeShowAdd = true)"
          :disabled="isCreateCustomer() ? false : true"
        >
          <span id="add" class="mx-2"><span class="size-16">Thêm</span></span>
        </button>
        <Add
          v-if="data.activeShowAdd"
          @refresh_customer="refresh_customer"
          :resetData="data.resetDataAdd"
        />
        <button
          type="button"
          class="btn btn-secondary ml-3"
          data-toggle="modal"
          data-target="#model-addHabit"
          @click="showAddHabit()"
          :disabled="isCreateHabit() ? false : true"
        >
          <span id="add" class="mx-2"
            ><span class="size-16">Thêm thói quen</span></span
          >
        </button>
        <AddHabit
          v-if="data.showAddHabit"
          :item="data.customerValue"
          :habitAdd="data.habitAdd"
          :eventAdd="data.eventAdd"
          @freshHabitAdd="(data.habitAdd = [{ name: '' }]), reFresh1()"
          @remove="removeItem"
        />
      </div>
    </div>
    <!-- Table -->
    <Table
      :items="setPages"
      :fields="[
        'Tên',
        'Email',
        'Sdt',
        'Công việc',
        'Công ty',
        'Loại khách hàng',
      ]"
      :selectAll="data.selectAll"
      :startRow="data.startRow"
      @selectAll="(value) => handleSelectAll(value)"
      @delete="handleDelete"
      @edit="edit"
      @view="view"
      :showActionList="[
        isReadCustomer() ? true : false,
        isEditCustomer() ? true : false,
        isDeleteCustomer() ? true : false,
      ]"
    />
    <!-- Pagination -->
    <Pagination
      :numberOfPages="data.numberOfPages"
      :totalRow="data.totalRow"
      :startRow="data.startRow"
      :endRow="data.endRow"
      :currentPage="data.currentPage"
      @update:currentPage="(value) => (data.currentPage = value)"
      class="mx-3"
    />
    <Edit
      v-if="data.activeShowEdit"
      :item="data.viewValue"
      :class="[data.activeEdit ? 'show-modal' : 'd-none']"
      @cancel="data.activeEdit = false"
      @refresh_customer="refresh_customer"
    />
    <View
      :item="data.viewValue"
      :itemViewCareCus="data.viewCareCus"
      :Events="data.Events"
    />
  </div>
</template>

<style scoped>
.border-box {
  border: 1px solid var(--gray);
  border-radius: 5px;
}

.menu {
  /* border: 1px solid var(--gray); */
  border-collapse: collapse;
}

.menu a {
  border: 1px solid var(--gray);
  border-collapse: collapse;
  padding: 8px 12px;
  font-size: 15px;
}

.active-menu {
  color: blue;
}

.none-active-menu {
  color: var(--dark);
}

.border-hr {
  border-top: 1px solid var(--gray);
}

#add,
#delete-all {
  font-size: 14px;
}
</style>
