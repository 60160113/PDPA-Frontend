<template>
  <div>
    <CCard>
      <CCardHeader
        ><strong class="text-primary"
          >รายงานการเข้าใช้งานระบบ</strong
        ></CCardHeader
      >
      <CCardBody>
        <CDataTable
          :items="list"
          :fields="fields"
          column-filter
          :items-per-page="5"
          :itemsPerPageSelect="{
            label: 'แสดง',
          }"
          :tableFilter="{
            label: 'ค้นหา: ',
            placeholder: 'ค้นหา',
          }"
          sorter
          :loading="isLoaded"
          hover
          striped
          border
          pagination
          cleaner
        >
          <template #time="{ item }">
            <td>
              {{ new Date(item.time).toLocaleDateString() }}
              {{ new Date(item.time).toLocaleTimeString() }}
            </td>
          </template>
          <template #action="{ item }">
            <td>
              <CBadge
                :color="item.action == 'Login' ? 'success' : 'danger'"
                class="mfs-auto"
                style="font-size: 15px"
              >
                {{ item.action }}
              </CBadge>
            </td>
          </template>
        </CDataTable>
      </CCardBody>
    </CCard>
  </div>
</template>

<script>
const fields = [
  { key: "user", label: "User" },
  { key: "time", label: "Time" },
  { key: "action", label: "Actions" },
];

export default {
  async created() {
    this.isLoaded = true;

    const loginReport = await this.$http.get(
      `${process.env.VUE_APP_ALFRESCO_SERVICES}audit/query/alfresco-access/alfresco-access/login/user?forward=false&verbose=true&limit=1000`
    );

    const logoutReport = await this.$http.get(
      `${process.env.VUE_APP_ALFRESCO_SERVICES}audit/query/alfresco-access/alfresco-access/logout/user?forward=false&verbose=true&limit=1000`
    );

    this.list = [...loginReport.data.entries, ...logoutReport.data.entries]
      .map((item) => {
        return {
          user: item.user,
          time: item.time,
          action:
            "/alfresco-access/login/user" in item.values ? "Login" : "Logout",
        };
      })
      .sort((a, b) => (new Date(a.time) > new Date(b.time) ? -1 : 1));
    this.isLoaded = false;
  },
  data() {
    return {
      list: [],
      fields,

      isLoaded: false,
    };
  },
};
</script>