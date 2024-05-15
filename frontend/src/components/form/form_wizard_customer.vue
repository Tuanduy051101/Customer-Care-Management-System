<script>
import {
  reactive,
  Customer_Types,
  onBeforeMount,
  http_getAll,
  alert_success,
  alert_error,
  http_create,
  http_deleteOne,
  alert_delete,
  Company_KH,
  Customer_Work,
  Customer,
  Event,
  http_getOne,
  watch,
} from "../../views/common/import";

import Swal from "sweetalert2";
import Select_Advanced from "./select_advanced.vue";

export default {
  components: {
    // Select_Advanced
    Select_Advanced,
  },
  props: {
    item: {
      type: Object,
      default: {},
    },
    id_form: {
      type: String,
      default: "model-form-wizard",
    },
    resetData: {
      type: Boolean,
      default: false,
    },
  },
  setup(props, context) {
    watch(
      () => props.resetData,
      async (newValue, oldValue) => {
       
        await refresh();
      },
      { immediate: true }
      //có props
    );
    // declare variables
    const data = reactive({
      imgSrc: null,
      stepList: [
        {
          _id: 1,
          name: "Thông tin",
        },
        {
          _id: 2,
          name: "Công việc",
        },
      ],
      activeStep: 1,
      dataSelect: null,
      items: null,
      modelValue: "--Chọn--",
      customer: null,
      event: null,
    });

    const viewData = reactive({
      customerInfo: {
        avatar: "",
        customerTypesId: 1,
        name: "",
        birthday: "",
        address: "",
        phone: "",
        email: "",
      },
      customerCompany: {
        name: "",
        _id: "",
      },
      customerWork: {
        customerId: "",
        current_workplace: "",
        work_history: "",
        current_position: "",
        work_temp: "",
        companyId: "",
      },
    });

    // refresh method
    const refresh = async () => {
      const cusTypes = await http_getAll(Customer_Types);
      const company = await http_getAll(Company_KH);
      const customer = await http_getAll(Customer);
      const event = await http_getAll(Event);
      data.items = [
        ...company.documents,
        {
          _id: "other",
          name: "khác",
        },
      ];
      data.dataSelect = cusTypes.documents;
      data.customer = customer.documents;
      data.event = event.documents;
    };

    // handle display img
    const onFileChange = (event) => {
      viewData.customerInfo.avatar = event.target.files[0];

      // handle display img
      const file = event.target.files[0];
      
      const reader = new FileReader();

      reader.onload = (event) => {
        data.imgSrc = event.target.result;
      };

      reader.readAsDataURL(file);
    };

    const handleAddCustometType = async () => {
      if (viewData.customerInfo.customerTypesId === "Add") {
        const { value: customerType, dismiss } = await Swal.fire({
          title: "Thêm loại khách hàng",
          input: "text",
          inputLabel: "Tên loại khách hàng",
          inputValidator: (value) => {
            if (!value) {
              return "Bạn không được phép để trường này trống!";
            }
          },
          showCancelButton: true,
          cancelButtonText: "Cancel",
          allowOutsideClick: false,
          didClose: () => {
            if (dismiss === "cancel") {
              viewData.customerInfo.customerTypesId = "1";
              refresh();
            }
          },
        });

        if (customerType) {
          const res = await http_create(Customer_Types, { name: customerType });
          if (res.error) {
            alert_error(`Thêm loại khách hàng`, `${res.msg}`);
          } else {
            viewData.customerInfo.customerTypesId = res.document._id;
            refresh();
            alert_success(
              `Thêm loại khách hàng`,
              `Loại khách hàng ${customerType} đã được tạo thành công.`
            );
          }
        }
      }
    };

    const searchSelect = async (value) => {
      await refresh(),
        (data.items = data.items.filter((value1, index) => {
          
          return value1.name.includes(value) || value.length == 0;
        }))
       
    };

    const choosed = async (_id) => {
      
      if (_id === "other") {
        const { value: companyName } = await Swal.fire({
          title: "Thêm công ty",
          input: "text",
          inputLabel: "Tên công ty",
          inputValidator: (value) => {
            if (!value) {
              return "Bạn không được phép để trường này trống!";
            }
          },
        });

        const res = await http_create(Company_KH, { name: companyName });
        if (res.error) {
          alert_error(`Lổi`, `${res.msg}`);
        } else {
          data.modelValue = companyName;
          refresh();
          viewData.customerCompany._id = res.document._id;
          
          alert_success(
            `Thành công`,
            `Công ty ${companyName}  đã được tạo thành công.`
          );
        }
      } else {
        const rs = await http_getOne(Company_KH, _id);
        data.modelValue = rs.name;
        viewData.customerCompany._id = _id;
        
      }
    };

    const deleted = async (item) => {
      const isConfirmed = await alert_delete(
        `Xoá`,
        `Bạn có chắc chắn muốn xoá công ty này không ?`
      );

      if (isConfirmed) {
        const result = await http_deleteOne(Company_KH, item._id);
        if (result.error) {
          alert_error("Lổi", result.msg);
        } else {
          alert_success(`Thành công`, result.msg);
          data.modelValue = "--Chọn--";
          refresh();
        }
      }
    };

    // handle create customer
    const create = async (event) => {
   
      event.preventDefault();
      let isCheck = false;
      refresh();
      for (const value of data.customer) {
        if (
          value.phone == viewData.customerInfo.phone &&
          value.email == viewData.customerInfo.email
        ) {
          isCheck = true;
        }
      }
      if (isCheck == true) {
        return alert_error(
          "Lổi",
          `Thông tin khách hàng ${viewData.customerInfo.name} đã có`
        );
      } else {
        const formData = new FormData();
        formData.append("avatar", viewData.customerInfo.avatar);
        formData.append("name", viewData.customerInfo.name);
        formData.append("birthday", viewData.customerInfo.birthday);
        formData.append("address", viewData.customerInfo.address);
        formData.append("phone", viewData.customerInfo.phone);
        formData.append("email", viewData.customerInfo.email);
        formData.append(
          "customerTypesId",
          viewData.customerInfo.customerTypesId
        );

        
        const res = await http_create(Customer, formData);

        if (res.error) {
          alert_error(`Lổi`, res.msg);
        } else {
          const object = {
            customerId: res.document._id,
            current_workplace: viewData.customerWork.current_workplace,
            work_history: viewData.customerWork.work_history,
            current_position: viewData.customerWork.current_position,
            work_temp: viewData.customerWork.work_temp,
            companyId: viewData.customerCompany._id,
          };

          const customerWork = await http_create(Customer_Work, object);
          
          if (customerWork.error) {
            alert_error(`Lổi`, customerWork.msg);
          } else {
            alert_success("Thành công", customerWork.msg);
            context.emit("refresh_customer");
          }
        }
      }
    };

    //   CustomerEvent,
    // CustomerHabit,
    // life cycle
    onBeforeMount(() => {
      refresh();
    });
    return {
      create,
      data,
      viewData,
      onFileChange,
      handleAddCustometType,
      searchSelect,
      choosed,
      deleted,
    };
  },
};
</script>

<template>
  <!-- The Modal -->
  <div class="modal" :id="id_form">
    <div class="modal-dialog modal-lg">
      <div class="modal-content">
        <!-- Modal Header -->
        <div class="modal-header">
          <h4 class="modal-title" style="font-size: 18px">Thêm Khách Hàng</h4>
          <button type="button" class="close" data-dismiss="modal">
            &times;
          </button>
        </div>

        <!-- Modal body -->
        <div class="model-body" style="min-height: 250px">
          <div class="d-flex">
            <!-- steps -->
            <div class="d-flex flex-column" style="height: 100%">
              <div
                class="d-flex mt-3 mx-3"
                v-for="(value, index) in data.stepList"
                :key="index"
              >
                <span
                  @click="data.activeStep = index + 1"
                  class="step-id px-3 py-2"
                  :class="[data.activeStep == index + 1 ? 'active-step' : '']"
                  >{{ value._id }}</span
                >
                <span
                  class="d-flex align-items-center px-3"
                  :class="[data.activeStep == index + 1 ? 'active-step' : '']"
                  >{{ value.name }}</span
                >
              </div>
            </div>
            <!-- form -->
            <div
              class="d-flex flex-grow-1 flex-column step-content px-3 my-3"
              style="width: 10000px"
            >
              <form
                action=""
                class="was-validated"
                style="width: 100%"
                method="post"
              >
                <!--begin  page 1 -->
                <div v-if="data.activeStep == 1" class="page-1">
                  <div class="form-row">
                    <div class="form-group col-md-6">
                      <label for="name"
                        >Tên (<span style="color: red">*</span>)</label
                      >
                      <input
                        type="text"
                        class="form-control"
                        id="name"
                        v-model="viewData.customerInfo.name"
                        required
                      />
                    </div>
                    <div class="form-group col-md-6">
                      <label for="birthdate"
                        >Ngày sinh (<span style="color: red">*</span>)</label
                      >
                      <input
                        type="date"
                        class="form-control"
                        id="birthdate"
                        v-model="viewData.customerInfo.birthday"
                        required
                      />
                    </div>
                  </div>
                  <div class="form-group">
                    <label for="address"
                      >Địa chỉ (<span style="color: red">*</span>)</label
                    >
                    <textarea
                      name=""
                      class="form-control"
                      v-model="viewData.customerInfo.address"
                      required
                    ></textarea>
                  </div>
                  <div class="form-group">
                    <label for="phone"
                      >Sdt (<span style="color: red">*</span>)</label
                    >
                    <input
                      type="text"
                      class="form-control"
                      id="phone"
                      v-model="viewData.customerInfo.phone"
                      required
                    />
                  </div>
                  <div class="form-row">
                    <div class="form-group col-md-6">
                      <label for="email"
                        >Email (<span style="color: red">*</span>)</label
                      >
                      <input
                        type="text"
                        class="form-control"
                        id="email"
                        v-model="viewData.customerInfo.email"
                        required
                      />
                    </div>
                    <div class="form-group col-md-6">
                      <label for="type"> Loại khách hàng </label>
                      <select
                        id="type"
                        class="form-control"
                        v-model="viewData.customerInfo.customerTypesId"
                        @click="handleAddCustometType"
                      >
                        <option value="1" style="">--Chọn--</option>
                        <option
                          v-for="(value, index) in data.dataSelect"
                          :key="index"
                          :value="value._id"
                        >
                          {{ value.name }}
                        </option>
                        <option value="Add">khác</option>
                      </select>
                    </div>
                  </div>
                  <div class="row">
                    <div class="col">
                      <label for="wor_work_history">Avatar</label>
                      <input
                        type="file"
                        class="form-control"
                        name="file"
                        id="imageUpload"
                        accept=".png, .jpg, jpeg"
                        :maxFileSize="1000000"
                        ref="fileInput"
                        @change="onFileChange"
                      />
                    </div>
                    <div class="col">
                      <img :src="data.imgSrc" alt="" height="100" />
                    </div>
                  </div>

                  <div class="form mt-3">
                    <div class="form-group">
                      <label for="wor_work_history">Công ty</label>
                      <Select_Advanced
                        style="height: 40px"
                        :modelValue="data.modelValue"
                        :options="data.items"
                        @searchSelect="searchSelect"
                        @delete="deleted"
                        @chose="choosed"
                      />
                    </div>
                  </div>
                </div>
                <!--end  page 1 -->

                <!--begin page 2 -->
                <div v-if="data.activeStep == 2" class="page-2">
                  <div class="form-row">
                    <div class="form-group col-md-6">
                      <label for="wor_current_workplace"
                        >Nơi đang công tác (<span style="color: red">*</span
                        >)</label
                      >
                      <input
                        type="text"
                        class="form-control"
                        id="wor_current_workplace"
                        v-model="viewData.customerWork.current_workplace"
                        required
                      />
                    </div>
                    <div class="form-group col-md-6">
                      <label for="wor_current_position"
                        >Chức vụ đang công tác (<span style="color: red">*</span
                        >)</label
                      >
                      <input
                        type="text"
                        class="form-control"
                        id="wor_current_position"
                        v-model="viewData.customerWork.current_position"
                        required
                      />
                    </div>
                  </div>

                  <div class="form-group">
                    <label for="wor_work_history"
                      >Lịch sử công tác (<span style="color: red">*</span
                      >)</label
                    >
                    <textarea
                      name=""
                      class="form-control"
                      v-model="viewData.customerWork.work_history"
                      required
                    ></textarea>
                  </div>
                  <div class="form-group">
                    <label for="wor_work_temp">Nhiệm kỳ</label>
                    <input
                      type="text"
                      class="form-control"
                      id="wor_work_temp"
                      v-model="viewData.customerWork.work_temp"
                      required
                      style="border-color: #28a745"
                    />
                  </div>
                </div>
                <!--end page 2 -->
                <button
                  type="submit"
                  v-if="data.activeStep == data.stepList.length"
                  @click="create"
                  class="btn btn-primary"
                >
                  Thêm
                </button>
              </form>

              <div class="d-flex justify-content-end mt-3">
                <span
                  v-if="
                    data.activeStep >= 1 &&
                    data.activeStep < data.stepList.length
                  "
                  class="btn-next d-flex align-items-center px-3 py-1 mx-2"
                  @click="data.activeStep = data.activeStep + 1"
                  >Tiếp tục
                  <span
                    class="material-symbols-outlined d-flex align-items-center"
                  >
                    navigate_next
                  </span>
                </span>
                <span
                  v-if="
                    data.activeStep > 1 &&
                    data.activeStep <= data.stepList.length
                  "
                  class="btn-prev d-flex align-items-center px-3 py-1"
                  @click="data.activeStep = data.activeStep - 1"
                  ><span
                    class="material-symbols-outlined d-flex align-items-center mx-2"
                  >
                    navigate_before </span
                  >Quay lại</span
                >
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.step-id {
  border: 1px solid var(--gray);
  border-radius: 5px;
  cursor: pointer;
}

.step-content {
  border-left: 1px solid var(--gray);
}

input {
  width: 100%;
}

.active-step {
  color: blue;
}

.btn-next {
  border: 1px solid var(--gray);
  border-radius: 5px;
  cursor: pointer;
}

.btn-next:hover {
  background-color: green;
  color: white;
}

.btn-prev {
  border: 1px solid var(--gray);
  border-radius: 5px;
  cursor: pointer;
}

.btn-prev:hover {
  background-color: red;
  color: white;
}
@media screen and (min-width: 739px) and (max-width: 992px) {
  .modal-content {
    width: 200%;
    margin-left: -50%;
  }
}
</style>
