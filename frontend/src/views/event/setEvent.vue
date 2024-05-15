<script>
import Customer_Table from "./customer.vue";
import {
  // components
  Table,
  Pagination,
  Dropdown,
  Select,
  Search,
  DeleteAll,
  Select_Advanced,
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
  // service
  Event,
  Habit,
  Account,
  Appointment,
  Center_VNPT,
  Company_KH,
  Customer_Types,
  Customer_Work,
  Customer,
  Cycle,
  Department,
  Employee,
  Log,
  Permission,
  Position,
  Role,
  Task,
  Unit,
  Customer_Event,
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
  alert_info,
} from "../common/import.js";
export default {
  props: {
    item: {
      type: Object,
      default: {},
    },
    refreshTable: {
      type: Boolean,
      default: false,
    },
  },
  components: {
    Select_Advanced,
    Customer_Table,
  },
  setup(props, ctx) {
    const data = reactive({
      customerList: [],
      customerId: "",
      eventId: "",
      customer_eventId: "",
      customer_eventList: [],
    });
    const create = async () => {
      try {
        let isSuccess = false;
        for (let value of data.customerList) {
          if (value.checked == true && !isStringFound(value.Customer._id)) {
            let result = await http_create(Customer_Event, {
              customerId: value.Customer._id,
              eventId: props.item._id,
            });
            
            if (result.error == true) {
              alert_error(
                `Áp dụng sự kiện`,
                `Đã áp dụng sự kiện ${props.item.name} với khách hàng ${value.Customer.name}`
              );
            } else {
              isSuccess = true;
            }
          } else if (value.checked == false) {
            if (isStringFound(value.Customer._id)) {
              await Customer_Event.update(data.customer_eventId, {
                EventId: data.eventId,
                CustomerId: data.customerId,
              });
              isSuccess = true;
            }
          }
        }
        if (isSuccess) {
          alert_success(
            `Áp dụng sự kiện`,
            `Bạn đã áp dụng sự kiện đối với khách hàng thành công thành công.`
          );
          ctx.emit("refresh1");
        }
        refresh();
        props.refreshTable = true;
      } catch (error) {
        console.log(error);
      }
    };
    
    const isStringFound = (_id) => {
      return data.customer_eventList.some((item) => {
        if (item.CustomerId.toString() == _id && item.EventId == props.item._id) {
         
          data.customerId = item.CustomerId;
          data.eventId = item.EventId;
          data.customer_eventId = data.customerId + data.eventId;
        }
        return item.CustomerId.toString() == _id && item.EventId == props.item._id;
      });
    };
    const refresh = async () => {
      data.customer_eventList = await http_getAll(Customer_Event);
    };
    onBeforeMount(async () => {
      // data.customerList = await http_getAll(Customer);
      refresh();
    });
    return {
      create,
      data,
      isStringFound,
    };
  },
};
</script>

<template>
  <!-- The Modal -->
  <div class="modal" id="model-setEvent">
    <div class="modal-dialog modal-lg">
      <div class="modal-content">
        <!-- Modal Header -->
        <div class="modal-header">
          <h4 class="modal-title" style="font-size: 18px">
            Thêm khách hàng áp dụng sự kiện
          </h4>
          <button type="button" class="close" data-dismiss="modal">&times;</button>
        </div>
        <!-- Modal body -->
        <div class="modal-body">
          <form action="" class="was-validated">
            <div class="form-group">
              <label for="name">Tên sự kiện(<span style="color: red">*</span>):</label>
              <input
                type="text"
                class="form-control"
                id="name"
                name="name"
                v-model="item.name"
                disabled
              />
            </div>
            <div class="form-group">
              <label for="content"
                >Nội dung sự kiện(<span style="color: red">*</span>):</label
              >
              <textarea
                id="content"
                disabled
                class="form-control"
                rows="5"
                v-model="item.content"
              ></textarea>
            </div>
            <!-- <div class="form-group">
              <label for="content"
                >Thời gian diễn ra sự kiện(<span style="color: red">*</span
                >):</label
              >
              <input
                type="datetime-local"
                class="form-control"
                id="time_duration"
                name="time_duration"
                v-model="item.time_duration"
                disabled
              />
            </div> -->
            <div class="form-group">
              <!-- <label for="content"
                >Thời gian diễn ra sự kiện(<span style="color: red">*</span
                >):</label
              > -->
              <div class="d-flex">
                <div class="form-group mr-3 w-100">
                  <label for="content"
                    >Thời gian bắt đầu(<span style="color: red">*</span>):</label
                  >
                  <input
                    type="datetime-local"
                    class="form-control w-100"
                    id="start_time"
                    name="start_time"
                    v-model="item.start_time"
                    disabled
                  />
                </div>
                <div class="form-group w-100">
                  <label for="content"
                    >Thời gian kết thúc(<span style="color: red">*</span>):</label
                  >
                  <input
                    type="datetime-local"
                    class="form-control mr-3 w-100"
                    id="end_time"
                    name="end_time"
                    v-model="item.end_time"
                    disabled
                  />
                </div>
              </div>
            </div>
            <div class="form-group">
              <label for="content">Địa điểm:</label>
              <textarea
                id="content"
                class="form-control"
                rows="2"
                v-model="item.place"
                disabled
              ></textarea>
            </div>
            <div class="form-group">
              <label for="content"
                >Khách hàng áp dụng(<span style="color: red">*</span>):</label
              >
              <div class="">
                <Customer_Table
                  :refreshTable="refreshTable"
                  :item="item"
                  :customer_eventListObject="data.customer_eventList"
                  @create="
                    (value) => {
                      data.customerList = value;
                      create();
                    }
                  "
                  @close="$emit('close', false)"
                />
              </div>
            </div>
            <!-- <button
              type="button"
              class="btn btn-primary px-3 py-2"
              style="font-size: 14px"
              @click="create"
              id="add"
            >
              <span>Thêm</span>
            </button> -->
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
@media screen and (min-width: 739px) and (max-width: 992px) {
  .modal-content {
    width: 200%;
    margin-left: -50%;
  }
}
</style>
