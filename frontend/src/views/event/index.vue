<script>
import Add from "./add.vue";
import AddAppointment from "../appointment/add.vue";
import Edit from "./edit.vue";
import View from "./view.vue";
import setEvent from "./setEvent.vue";
import FormWizard from "../../components/form/form-wizard.vue";
import {
  // components
  Table,
  Pagination,
  Dropdown,
  Select,
  Search,
  DeleteAll,
  Select_Advanced,
  Input_Datetime,
  // compositions
  reactive,
  computed,
  watch,
  ref,
  onBeforeMount,
  // router
  useRouter,
  // format date or datetime
  formatDateTime,
  formatDate,
  formatDateTime_2,
  // service
  Event,
  // http service
  http_getAll,
  http_create,
  http_getOne,
  http_deleteOne,
  http_update,
  // alert
  alert_success,
  alert_error,
  alert_delete,
  alert_warning,
  alert_delete_wide,
} from "../common/import.js";

import {
  isDeleteEvent,
  isEditEvent,
  isCreateEvent,
  isReadEvent,
  isSetEvent,
} from "../../use/getSessionItem";

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
    Select_Advanced,
    FormWizard,
    setEvent,
    AddAppointment,
    Input_Datetime,
  },
  setup(ctx) {
    const data = reactive({
      items: [],
      entryValue: 5,
      numberOfPages: 1,
      totalRow: 0,
      startRow: 0,
      endRow: 0,
      currentPage: 1,
      searchText: "",
      itemAdd: {
        name: "",
        content: "",
        time_duration: "",
        end_time: "",
        start_time: "",
        place: "",
      },
      activeEdit: false,
      editValue: {},
      searchSelect: "",
      optionSelect: [
        {
          _id: 1,
          name: "1",
        },
        {
          _id: 2,
          name: "2",
        },
      ],
      test: {
        a: "",
        b: "",
      },
      eventValue: {},
      showSetEvent: false,
      choseSearch: "",
      selectAll: [
        {
          checked: false,
        },
      ],
      startTimeValue: "",
      endTimeValue: "",
      refreshTable: false,
      viewValue: {},
      showView: false,
    });
    const toString = computed(() => {
      if (data.choseSearch == "name") {
        return data.items.map((value, index) => {
          return [value.name].join("").toLocaleLowerCase();
        });
      } else if (data.choseSearch == "place") {
        return data.items.map((value, index) => {
          return [value.place].join("").toLocaleLowerCase();
        });
      } else if (data.choseSearch == "amountCustomer") {
        return data.items.map((value, index) => {
          return [value.totalCustomer].join("").toLocaleLowerCase();
        });
      } else if (data.choseSearch == "customer") {
        return data.items.map((value, index) => {
          return [
            value.Customers.map((Customer) => Customer.name).join(""),
            value.Customers.map((Customer) => Customer.phone).join(""),
            value.Customers.map((Customer) => Customer.email).join(""),
          ]
            .join("")
            .toLocaleLowerCase();
        });
      }
      return data.items.map((value, index) => {
        return [value.name, value.place].join("").toLocaleLowerCase();
      });
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

    // routers
    const router = useRouter();

    // watch
    const activeMenu = ref(1);
    watch(activeMenu, (newValue, oldValue) => {
      router.push({ name: "Habit" });
    });

    // methods
    const create = async () => {
      data.itemAdd.time_duration = [
        data.itemAdd.start_time,
        data.itemAdd.end_time,
      ].join(" to ");
      const result = await http_create(Event, data.itemAdd);
      if (!result.error) {
        alert_success(
          `Thêm sự kiện`,
          `Sự kiện ${result.document.name} lúc ${formatDateTime_2(
            result.document.time_duration
          )} đã được tạo thành công.`
        );
        refresh();
      } else if (result.error) {
        alert_error(`Thêm sự kiện`, `${result.msg}`);
      }
    };
    const update = async (item) => {
      const result = await http_update(Event, editValue._id, editValue);
      if (!result.error) {
        alert_success(`Sửa sự kiện`, `${result.msg}`);
        refresh();
      } else if (result.error) {
        alert_error(`Sửa sự kiện`, `${result.msg}`);
      }
    };

    const setEvent = () => {
      data.eventValue = {};
      data.showSetEvent = false;
      for (let value of data.items) {
        if (value.checked == true) {
          data.eventValue = value;
          data.eventValue.time_duration =
            data.eventValue.time_duration.toUpperCase();
          data.showSetEvent = true;
          break;
        }
      }
      if (data.showSetEvent == false) {
        alert_warning(
          `Thêm khách hàng áp dụng sự kiện`,
          `Vui lòng chọn sự kiện.`
        );
      }
    };

    const setEvent1 = () => {
      data.eventValue = {};
      data.showSetEvent = false;
      for (let value of data.items) {
        if (value.checked == true) {
          data.eventValue = value;
          data.eventValue.time_duration =
            data.eventValue.time_duration.toUpperCase();
          data.showSetEvent = true;
          break;
        }
      }
    };

    const deleteOne = async (_id) => {
      const event = await http_getOne(Event, _id);

      const isConfirmed = await alert_delete(
        `Xoá sự kiện`,
        `Bạn có chắc chắn muốn xoá sự kiện ${event.name} lúc ${formatDateTime_2(
          event.time_duration
        )} không ?`
      );

      if (isConfirmed == true) {
        const result = await http_deleteOne(Event, _id);
        alert_success(
          `Xoá sự kiện`,
          `Bạn đã xoá thành công sự kiện ${
            result.document.name
          } lúc ${formatDateTime_2(result.document.time_duration)}.`
        );
        refresh();
      }
    };

    const deleteMany = async () => {
      try {
        const deleteArray = data.items.filter((value, index) => {
          return value.checked == true;
        });
        let name, phone, email;
        let contentAlert = `<p>Bạn có muốn xoá tất cả các sự kiện này không?</p><p>Tổng số sự kiện sẽ xoá là: <span style="color: blue;">${deleteArray.length}</span></p>
          <table class="table table-bordered">
      <thead>
        <tr>
          <th>Tên sự kiện</th>
          <th>Ngày diễn ra sự kiện</th>
          <th>Địa điểm</th>
        </tr>
      </thead> <tbody>`;

        for (let value of deleteArray) {
          contentAlert += `<tr>
          <td>${value.name}</td>
          <td>${value.time_duration_format}</td>
          <td>${value.place != null ? value.place : "không có"}</td>
        </tr>`;
        }
        contentAlert += `</tbody>
    </table>`;
        const isConfirmed = await alert_delete_wide(
          `Xoá nhiều sự kiện`,
          contentAlert
        );
        if (isConfirmed) {
          let checkDeleteAll = false;
          for (let valueDelete of deleteArray) {
            const result = await http_deleteOne(Event, valueDelete._id);
            if (result.error) {
              alert_error("Lổi ", result.msg);
              checkDeleteAll = false;
            } else {
              checkDeleteAll = true;
            }
          }
          if (checkDeleteAll) {
            refresh();
            alert_success("Thành công", "Xóa sự kiện thành công");
          }
        }
      } catch (error) {
        console.log(error);
      }
    };

    const edit = async (editValue) => {
      editValue.time_duration = [editValue.start_time, editValue.end_time].join(
        " to "
      );
      const result = await http_update(Event, editValue._id, editValue);
      if (!result.error) {
        alert_success(`Sửa sự kiện`, `${result.msg}`);
        refresh();
      } else if (result.error) {
        alert_error(`Sửa sự kiện`, `${result.msg}`);
      }
    };

    const view = async (item) => {
      await refresh();
      item.start_time = item.time_duration.split(" to ")[0].toUpperCase();
      item.end_time = item.time_duration.split(" to ")[1].toUpperCase();
      item.time_duration_format = [
        formatDateTime(item.start_time),
        formatDateTime(item.end_time),
      ].join(" đến ");
      // item.time_duration = item.time_duration.toUpperCase();
      data.viewValue = item;
      data.showView = true;
      // router.push({ name: "Event.view", params: { id: _id } });
    };

    const getOne = async (_id) => {
      try {
        const result = await http_getOne(Event, _id);
        return result;
      } catch (error) {}
    };

    const refresh = async () => {
      data.items = await http_getAll(Event);
      for (const value of data.items) {
        value.start_time = value.time_duration.split(" to ")[0].toUpperCase();
        value.end_time = value.time_duration.split(" to ")[1].toUpperCase();
        value.time_duration_format = [
          formatDateTime(value.start_time),
          formatDateTime(value.end_time),
        ].join(" đến ");
        value.place = value.place != null ? value.place : "không có";
      }
      for (let value of data.items) {
        value.checked = false;
        value.totalCustomer = value.Customers.length;
      }

      // filter
      if (data.startTimeValue.length > 0) {
        if (data.endTimeValue.length == 0) {
          data.items = data.items.filter((value, index) => {
            return value.start_time == data.startTimeValue.toLocaleLowerCase();
          });
        } else {
          data.items = data.items.filter((value, index) => {
            return (
              new Date(value.start_time.toUpperCase()) >=
                new Date(data.startTimeValue) &&
              new Date(value.start_time.toUpperCase()) <=
                new Date(data.endTimeValue)
            );
          });
        }
      } else if (data.endTimeValue.length > 0) {
        data.items = data.items.filter((value, index) => {
          return value.end_time == data.endTimeValue.toLocaleLowerCase();
        });
      }
    };

    const refresh1 = async () => {
      const events = await http_getAll(Event);
      for (var i = 0; i < events.length; i++) {
        data.items[i].totalCustomer = events[i].Customers.length;
        data.items[i].Customers = events[i].Customers;
        // data.items[i].checked = false;
      }
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

    const delete_a = async (objectData) => {
      // console.log("delete_a", objectData);
    };

    // handle http methods

    // Hàm callback được gọi trước khi component được mount (load)
    onBeforeMount(async () => {
      refresh();
    });

    return {
      data,
      setPages,
      activeMenu,
      create,
      update,
      deleteOne,
      edit,
      view,
      delete_a,
      refresh,
      getOne,
      setEvent,
      handleSelectAll,
      deleteMany,
      setEvent,
      setEvent1,
      refresh1,
      isDeleteEvent,
      isEditEvent,
      isCreateEvent,
      isReadEvent,
      isSetEvent,
    };
  },
};
</script>

<template>
  <div class="border-box d-flex flex-column ml-2">
    <!-- Menu -->
    <div class="d-flex menu my-3 mx-3 justify-content-end">
      <router-link
        :to="{ name: 'Event' }"
        @click="activeMenu = 1"
        :class="[activeMenu == 1 ? 'active-menu' : 'none-active-menu']"
      >
        <span class="size-17">Sự kiện</span>
      </router-link>
      <router-link
        :to="{ name: 'Habit' }"
        @click="activeMenu = 2"
        :class="[activeMenu == 2 ? 'active-menu' : 'none-active-menu']"
      >
        <span class="size-18">Thói quen</span>
      </router-link>
    </div>
    <!-- Filter -->
    <div class="border-hr mb-3"></div>
    <div class="d-flex flex-column">
      <span class="mx-3 mb-3 h6">Lọc sự kiện</span>
      <div class="d-flex mx-3">
        <div class="form-group w-100">
          <Input_Datetime
            :title="`Thời gian bắt đầu`"
            :entryValue="data.startTimeValue"
            @update:entryValue="
              (value) => (
                (data.startTimeValue = value), (data.currentPage = 1), refresh()
              )
            "
            @refresh="
              (data.startTimeValue = ''), refresh(), (data.currentPage = 1)
            "
            style="height: 35px"
          />
        </div>
        <div class="d-flex form-group w-100 ml-3">
          <Input_Datetime
            :title="`Thời gian kết thúc`"
            :entryValue="data.endTimeValue"
            @update:entryValue="
              (value) => ((data.endTimeValue = value), refresh())
            "
            @refresh="
              (data.endTimeValue = ''), refresh(), (data.currentPage = 1)
            "
            style="height: 35px"
          />
        </div>

        <!-- <div class="form-group ml-3">
        </div> -->
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
              name: 2,
              value: 2,
            },
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
              name: 'Tìm kiếm theo tên sự kiện',
            },
            {
              _id: 'place',
              name: 'Tìm kiếm địa điểm sự kiện',
            },
            {
              _id: 'amountCustomer',
              name: 'Tìm kiếm theo số lượng khách hàng',
            },
            {
              _id: 'customer',
              name: 'Tìm kiếm theo khách hàng (tên, sđt, email)',
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
          :disabled="isDeleteEvent() ? false : true"
        >
          <span id="delete-all" class="mx-2">Xoá</span>
        </button>
        <!-- <DeleteAll :items="data.items" /> -->
        <button
          type="button"
          class="btn btn-primary"
          data-toggle="modal"
          data-target="#model-add"
          :disabled="isCreateEvent() ? false : true"
        >
          <span id="add" class="mx-2">Thêm</span>
        </button>
        <Add :item="data.itemAdd" @create="create" />
        <button
          type="button"
          class="btn btn-secondary ml-3"
          data-toggle="modal"
          data-target="#model-setEvent"
          @click="setEvent()"
          :disabled="isSetEvent() ? false : true"
        >
          <span id="add" class="mx-2">Áp dụng</span>
        </button>
        <setEvent
          :refreshTable="data.refreshTable"
          v-if="data.showSetEvent"
          :item="data.eventValue"
          @refresh1="refresh1()"
          @close="data.showSetEvent = false"
        />
      </div>
    </div>
    <!-- Table -->
    <Table
      :items="setPages"
      :fields="[
        'Tên sự kiện',
        'Nội dung sự kiện',
        'Thời gian diễn ra',
        'Địa điểm',
        'Khách hàng',
      ]"
      :labels="[
        'name',
        'content',
        'time_duration_format',
        'place',
        'totalCustomer',
      ]"
      @delete="(value) => deleteOne(value)"
      :startRow="data.startRow"
      :selectAll="data.selectAll"
      @selectAll="(value) => handleSelectAll(value)"
      @refresh_event="((value) => (data.refreshTable = !value), setEvent1())"
      @edit="
        async (value, value1) => (
          (data.editValue = await getOne(value._id)),
          (data.editValue.start_time = data.editValue.time_duration
            .split(' to ')[0]
            .toUpperCase()),
          (data.editValue.end_time = data.editValue.time_duration
            .split(' to ')[1]
            .toUpperCase()),
          (data.activeEdit = value1),
          (data.editValue.time_duration =
            data.editValue.time_duration.toUpperCase())
        )
      "
      @view="(value) => view(value)"
      :showActionList="[
        isReadEvent() ? true : false,
        isEditEvent() ? true : false,
        isDeleteEvent() ? true : false,
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
  </div>
  <Edit
    :item="data.editValue"
    :class="[data.activeEdit ? 'show-modal' : 'd-none']"
    @cancel="data.activeEdit = false"
    @edit="edit(data.editValue)"
  />
  <View
    v-if="data.showView"
    :item="data.viewValue"
    @close="data.showView == false"
  />
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
.input {
  background-color: inherit;
  border: 1px solid var(--gray);
  border-radius: 5px;
}
@media screen and (max-width: 992px) {
  .select {
    width: 90px;
  }
  .search {
    width: 210px;
    margin-left: 2px !important ;
    margin-right: 2px;
  }
  .border-box {
    width: 100%;
    margin-left: 10px;
  }
}
</style>
