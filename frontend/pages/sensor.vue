<template>
  <div>
    <h6 class="text-muted">KELOLA SITE</h6>

    <el-form inline class="text-right" @submit.native.prevent="() => { return }">
      <el-form-item>
        <el-button
          icon="el-icon-plus"
          type="primary"
          size="small"
          @click="() => { showForm = true; selectedData = {}; }"
        >TAMBAH SITE</el-button>
      </el-form-item>
      <el-form-item>
        <el-input
          prefix-icon="el-icon-search"
          clearable
          size="small"
          placeholder="Cari"
          v-model="keyword"
          @change="(v) => { keyword = v; requestData(); }"
        ></el-input>
      </el-form-item>
    </el-form>

    <el-table
      @sort-change="sortChange"
      @filter-change="(f) => { let c = Object.keys(f)[0]; filters[c] = Object.values(f[c]); page = 1; requestData(); }"
      :default-sort="{prop: sort, order: order}"
      :data="tableData.data"
      stripe
      height="calc(100vh - 240px)"
    >
      <el-table-column label="#" type="index" :index="tableData.from"></el-table-column>
      <el-table-column
        label="Perusahaan"
        prop="perusahaan"
        :filters="filterPerusahaan"
        column-key="perusahaan_id"
      ></el-table-column>
      <el-table-column label="Site" prop="site" :filters="filterSite" column-key="site_id"></el-table-column>
      <el-table-column label="Kode" prop="kode"></el-table-column>
      <el-table-column label="Nama" prop="nama"></el-table-column>
      <el-table-column label="Keterangan" prop="keterangan"></el-table-column>
      <el-table-column label="Satuan" prop="satuan"></el-table-column>
      <el-table-column label="Nilai Minimum" prop="nilai_minimum"></el-table-column>
      <el-table-column label="Nilai Maximum" prop="nilai_maximum"></el-table-column>
      <el-table-column label="Nilai Rendah" prop="nilai_rendah"></el-table-column>
      <el-table-column label="Nilai Tinggi" prop="nilai_tinggi"></el-table-column>
      <el-table-column label="Jenis Chart" prop="jenis_chart"></el-table-column>
      <el-table-column
        label="Status"
        prop="status"
        :filters="statusFilter"
        :filter-multiple="false"
        column-key="status"
        width="100"
        align="center"
        header-align="center"
      >
        <template slot-scope="scope">
          <el-tag
            effect="dark"
            :type="scope.row.status ? 'success' : 'danger'"
            size="small"
          >{{scope.row.status ? 'Aktif' : 'Non-Aktif'}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column fixed="right" width="40px" align="center" header-align="center">
        <template slot="header">
          <el-button
            type="text"
            @click="() => { page = 1; keyword = ''; requestData(); }"
            icon="el-icon-refresh"
          ></el-button>
        </template>
        <template slot-scope="scope">
          <el-dropdown>
            <span class="el-dropdown-link">
              <i class="el-icon-more rotate-90"></i>
            </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item
                icon="el-icon-edit-outline"
                @click.native.prevent="() => { selectedData = scope.row; showForm = true; }"
              >Edit</el-dropdown-item>
              <el-dropdown-item
                icon="el-icon-delete"
                @click.native.prevent="deleteData(scope.row.id)"
              >Hapus</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </template>
      </el-table-column>
    </el-table>

    <div class="d-flex mt-3">
      <el-pagination
        class="flex-grow-1"
        background
        @current-change="(p) => { page = p; requestData(); }"
        @size-change="(s) => { pageSize = s; requestData(); }"
        layout="total, sizes, prev, pager, next"
        :page-size="pageSize"
        :page-sizes="[10, 25, 50, 100]"
        :total="tableData.total"
      ></el-pagination>
      <div class="text-right">
        <small>Menampilkan {{tableData.from}} - {{tableData.to}} dari {{tableData.total}}</small>
      </div>
    </div>

    <FormSensor
      :data="selectedData"
      :show="showForm"
      @refresh="requestData"
      @close="showForm = false"
    />
  </div>
</template>

<script>
import { mapState } from 'vuex'

export default {
  data() {
    return {
      tableData: {},
      filters: {},
      sort: 'nama',
      order: 'ascending',
      page: 1,
      pageSize: 10,
      keyword: '',
      selectedData: {},
      showForm: false
    }
  },
  computed: {
    ...mapState(['statusFilter', 'filterPerusahaan', 'filterSite'])
  },
  methods: {
    requestData() {
      const params = {
        pageSize: this.pageSize,
        page: this.page,
        keyword: this.keyword,
        sort: this.sort,
        order: this.order
      };

      this.$axios.get('/api/sensor', { params: Object.assign(params, this.filters) }).then(r => {
        this.tableData = r.data
      })
    },
    deleteData(id) {
      this.$confirm('Anda yakin akan menghapus data ini?', 'Peringatan', { type: 'warning'}).then(() => {
        this.$axios.delete(`/api/sensor/${id}`).then(r => {
          this.$message({
            message: r.data.message,
            type: 'success',
            showClose: true,
            duration: 10000
          });
          this.requestData()
        }).catch(e => {
          this.$message({
            message: e.response.data.message,
            type: 'error',
            showClose: true,
            duration: 10000
          })
        })
      }).catch(e => console.log(e))
    },
    sortChange(c) {
      if (c.prop != this.sort || c.order != this.order) {
        this.sort = c.prop;
        this.order = c.order;
        this.requestData();
      }
    },
  },
  created() {
    this.requestData()
  }
}
</script>
