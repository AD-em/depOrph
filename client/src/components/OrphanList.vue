<template>
  <b-container>
    <!-- row for table top controls -->
    <b-row>
      <!-- first column for [filter text input] -->
      <b-col lg="6" class="my-1">
        <b-form-group
          label="Filter"
          label-for="filter-input"
          label-cols-sm="3"
          label-align-sm="right"
          label-size="sm"
          class="mb-0"
        >
          <b-input-group size="sm">
            <b-form-input
              id="filter-input"
              v-model="filter"
              type="search"
              placeholder="Type to Search"
            ></b-form-input>

            <b-input-group-append>
              <b-button :disabled="!filter" @click="filter = ''"
                >Clear</b-button
              >
            </b-input-group-append>
          </b-input-group>
        </b-form-group>
      </b-col>

      <!-- second column for [filter by options{name, age, gender, sponsorshipStatus}] -->
      <b-col lg="6" class="my-1">
        <b-form-group
          v-model="sortDirection"
          label="Filter By"
          description="Leave all unchecked to filter on all data"
          label-cols-sm="2"
          label-align-sm="right"
          label-size="sm"
          class="mb-0"
          v-slot="{ ariaDescribedby }"
        >
          <b-form-checkbox-group
            v-model="filterOn"
            :aria-describedby="ariaDescribedby"
            class="mt-1"
          >
            <b-form-checkbox value="Name">Name</b-form-checkbox>
            <b-form-checkbox value="Age">Age</b-form-checkbox>
            <b-form-checkbox value="Gender">Gender</b-form-checkbox>
            <b-form-checkbox value="SponsorshipStatus"
              >Sponsorship Status</b-form-checkbox
            >
          </b-form-checkbox-group>
        </b-form-group>
      </b-col>
    </b-row>
    <!-- card with table & bottom controls -->
    <b-card class="b-table-card-body mb-3">
      <!-- orphans list lable -->
      <b-table
        hover
        striped
        bordered
        small
        responsive
        sticky-header="800px"
        show-empty
        :items="orphans"
        :fields="fields"
        :filter="filter"
        :filter-included-fields="filterOn"
        :sort-desc.sync="sortDesc"
        :current-page="currentPage"
        :per-page="perPage"
        @filtered="onFiltered"
      >
        <!-- what to show inside the show_details column -->
        <template #cell(show_details)="row">
          <b-button pill size="sm" @click="row.toggleDetails" class="mr-2">
            {{ row.detailsShowing ? "Hide" : "Show" }} Details
          </b-button>
        </template>

        <!-- collapsable details card that shows each row's additional details -->
        <template #row-details="row">
          <b-card>
            <!-- first row for mother's fullname and phone number -->
            <b-row class="mb-2">
              <b-col sm="3" class="text-sm-right"><b>Mother Name:</b></b-col>
              <b-col class="p-0 text-left">{{ row.item.MotherName }}</b-col>
              <b-col sm="3" class="text-sm-right"><b>Mother Mobile:</b></b-col>
              <b-col class="p-0 text-left">{{ row.item.MotherMobile }}</b-col>
            </b-row>

            <!-- second row for guardian's fullname and phone number -->
            <b-row class="mb-2">
              <b-col sm="3" class="text-sm-right"><b>Guardian Name:</b></b-col>
              <b-col class="p-0 text-left">{{ row.item.GuardianName }}</b-col>
              <b-col sm="3" class="text-sm-right"
                ><b>Guardian Mobile:</b></b-col
              >
              <b-col class="p-0 text-left">{{ row.item.GuardianMobile }}</b-col>
            </b-row>
            <b-button
              pill
              class="float-right"
              size="sm"
              @click="gotoProfile(row)"
              >View Profile</b-button
            >
          </b-card>
        </template>
      </b-table>

      <!-- table bottom controls -->
      <b-row>
        <!-- first column for [items/rows per page{select}] -->
        <b-col sm="5" md="6" class="mt-1">
          <b-form-group
            label="Items per page"
            label-for="per-page-select"
            label-cols-sm="6"
            label-cols-md="4"
            label-cols-lg="3"
            label-align-sm="right"
            label-size="sm"
            class="mb-0"
          >
            <b-form-select
              id="per-page-select"
              v-model="perPage"
              :options="pageOptions"
              size="sm"
            ></b-form-select>
          </b-form-group>
        </b-col>

        <!-- second column for [table pagination] -->
        <b-col sm="7" md="5" class="mt-1">
          <b-pagination
            v-model="currentPage"
            :total-rows="totalRows"
            :per-page="perPage"
            align="fill"
            size="sm"
            class="my-0"
          ></b-pagination>
        </b-col>
      </b-row>
    </b-card>
  </b-container>
</template>

<script>
import axios from "axios";
import moment from "moment";
export default {
  data() {
    return {
      orphans: [],
      fields: [
        { key: "id", label: "ID", sortable: true, class: "text-left" },
        { key: "Name", label: "Name", sortable: true, class: "text-left" },
        { key: "Age", label: "Age", sortable: true },
        { key: "Gender", label: "Gender", sortable: true },
        {
          key: "SponsorshipStatus",
          label: "Sponsorship Status",
          sortable: true
        },
        { key: "Show_details", label: "Show Details", sortable: false }
      ],
      pageOptions: [10, 15, 18, 20, 25, 50, 100],
      perPage: 18,
      currentPage: 1,
      totalRows: null,
      filter: null,
      filterOn: [],
      sortDirection: "asc",
      sortDesc: false
    };
  },
  computed: {
    // unnecessary computed property;
    // 😅😅😅forgot to remove it when I copy-pasted code from th docs😅😅😅
    sortOptions() {
      // Create an options list from our fields
      return this.fields
        .filter(f => f.sortable)
        .map(f => {
          return { text: f.label, value: f.key };
        });
    }
  },
  mounted() {
    this.totalRows = this.orphans.length;
  },
  methods: {
    onFiltered(filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length;
      this.currentPage = 1;
    },
    gotoProfile(row) {
      this.$router.push({ name: "OrphanProfile", params: { id: row.item.id } });
    }
  },
  async created() {
    // on the created hook, request data from
    // the server and populate the orphans data property array
    await axios
      .post("/graphql/", {
        query: `{
                allOrphans(take: 100, orderBy: { created_at: desc }){
                    id
                    firstName
                    fatherName
                    grandFatherName
                    greatGrandFatherName
                    gender
                    dateOfBirth
                    sponsorshipStatus
                    mother{
                      firstName
                      middleName
                      lastName
                      phoneNumber
                    }
                    guardian{
                      firstName
                      middleName
                      lastName
                      mobile
                    }
                }
            }`
      })
      .then(
        res =>
          (this.orphans = res.data.data.allOrphans.map(val => ({
            id: val.id,
            Name: `${val.firstName} ${val.fatherName} ${val.grandFatherName} ${val.greatGrandFatherName}`,
            Age:
              moment().diff(val.dateOfBirth, "years") > 0
                ? `${moment().diff(val.dateOfBirth, "years")} years`
                : `${moment().diff(val.dateOfBirth, "months")} months` > 0
                ? moment().diff(val.dateOfBirth, "months")
                : `${moment().diff(val.dateOfBirth, "days")} days`,
            Gender: val.gender,
            SponsorshipStatus:
              val.sponsorshipStatus == "inProgress"
                ? "In-Progress"
                : val.sponsorshipStatus.charAt(0).toUpperCase() +
                  val.sponsorshipStatus.slice(1),
            MotherName: val.mother
              ? `${val.mother.firstName} ${val.mother.middleName} ${val.mother.lastName}`
              : "",
            MotherMobile:
              val.mother && val.mother.phoneNumber
                ? val.mother.phoneNumber
                : "N/A",
            GuardianName: val.guardian
              ? `${val.guardian.firstName} ${val.guardian.middleName} ${val.guardian.lastName}`
              : "",
            GuardianMobile:
              val.guardian && val.guardian.mobile ? val.guardian.mobile : "N/A",
            _cellVariants: {
              SponsorshipStatus:
                val.sponsorshipStatus == "inProgress"
                  ? "warning"
                  : val.sponsorshipStatus == "active"
                  ? "info"
                  : val.sponsorshipStatus == "suspended"
                  ? "danger"
                  : "success"
            }
          })))
      );
  }
};
</script>

<style scoped>
.b-table-card-body > .card-body {
  padding: 0px;
}
</style>
