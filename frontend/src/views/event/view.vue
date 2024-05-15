<script>
import { reactive } from "vue";
import Table_Customer from "./view_customer.vue";
export default {
  props: {
    item: {
      type: Object,
      default: {},
    },
  },
  components: {
    Table_Customer,
  },
  setup(props, ctx) {
    const data = reactive({});
    return {};
  },
};
</script>

<template>
  <div class="modal" id="model-view">
    <div class="modal-dialog modal-lg">
      <div class="modal-content">
        <!-- Modal Header -->
        <div class="modal-header">
          <h4 class="modal-title" style="font-size: 18px">
            Thông tin chi tiết sự kiện 
          </h4>
          <button type="button" class="close" data-dismiss="modal">
            &times;
          </button>
        </div>

        <!-- Modal body -->
        <div class="modal-body">
          <form class="was-validated">
            <div class="form-group">
              <label for="name">Tên(<span style="color: red">*</span>):</label>
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
                >Nội dung(<span style="color: red">*</span>):</label
              >
              <textarea
                id="content"
                disabled
                class="form-control"
                rows="3"
                v-model="item.content"
              ></textarea>
            </div>
            <div class="form-group">
              <!-- <label for="content"
                >Thời gian diễn ra sự kiện(<span style="color: red">*</span
                >):</label
              > -->
              <div class="d-flex">
                <div class="form-group mr-3 w-100">
                  <label for="content"
                    >Thời gian bắt đầu(<span style="color: red">*</span
                    >):</label
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
                    >Thời gian kết thúc(<span style="color: red">*</span
                    >):</label
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
              <label for="content"
                >Địa điểm:</label
              >
              <textarea
                id="content"
                disabled
                class="form-control"
                rows="2"
                v-model="item.place"
              ></textarea>
            </div>
          </form>
          <div class="form-group">
            <label for="duration">Danh sách khách hàng tham gia sự kiện:</label>
            <Table_Customer :customer_event="item.Customers" @close="$emit('close', false)" />
          </div>
          <div class="mt-2"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
label {
  font-size: 18px;
}
@media screen and (min-width: 739px) and (max-width: 992px) {
  .modal-content {
    width: 200%;
    margin-left: -50%;
  }
}
</style>
