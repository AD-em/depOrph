<template>
  <b-form>
    <b-form-text class="text-left mb-3"
      >The inputs marked by * are all required</b-form-text
    >

    <!-- physical health radios -->
    <b-form-row>
      <b-col class="text-left">
        <b-form-group
          label="Physical Health*"
          invalid-feedback="Please select one"
          :state="orphanPhysicalHealthState"
        >
          <b-form-radio
            name="physicalHealthRadios"
            size="sm"
            plain
            class="ml-2"
            value="Healthy"
            :state="orphanPhysicalHealthState"
            v-model="tmpOrphanPhysicalHealthRadio"
            >Healthy</b-form-radio
          ><b-input-group class="mt-2">
            <b-input-group-prepend is-text>
              <b-form-radio
                name="physicalHealthRadios"
                size="sm"
                plain
                class="pl-2 ml-1"
                value="Disabled"
                :state="orphanPhysicalHealthState"
                v-model="tmpOrphanPhysicalHealthRadio"
              />
              <label
                class="form-check-label mb-0 ml-1"
                for="physicalHealth-disabledRadio"
                >Disabled</label
              >
            </b-input-group-prepend>
            <b-form-input
              for="physicalHealth-disabledRadio"
              placeholder="Mention disability ... "
              :state="tmpOrphanDisabilityState"
              v-model="tmpOrphanDisability"
            ></b-form-input>
            <b-form-invalid-feedback :state="tmpOrphanDisabilityState">
              Please mention disbalility</b-form-invalid-feedback
            >
            <!-- 🔼:✅ TODO:DONE format the disabled as "disabled:disability" 🔼 -->
          </b-input-group>
        </b-form-group>
      </b-col>
    </b-form-row>

    <!-- psychological health radios + input -->
    <b-form-row>
      <b-col class="text-left">
        <b-form-group
          label="Psychological Health*"
          :state="orphanPsychologicalHealthState"
          invalid-feedback="Please select one"
        >
          <b-form-radio
            name="psychologicalHealthRadios"
            size="sm"
            plain
            value="Sociable"
            :state="orphanPsychologicalHealthState"
            v-model="orphanPsychologicalHealth"
            >Sociable</b-form-radio
          >
          <b-form-radio
            name="psychologicalHealthRadios"
            size="sm"
            plain
            value="Not Sociable"
            :state="orphanPsychologicalHealthState"
            v-model="orphanPsychologicalHealth"
            >Not Sociable</b-form-radio
          >
        </b-form-group></b-col
      >
    </b-form-row>

    <!-- other health issues textarea -->
    <b-form-row>
      <b-col class="text-left">
        <b-form-group
          label-for="other-health-issues-textarea"
          label="Other Health Issues"
        >
          <b-textarea
            id="other-health-issues-textarea"
            class="mt-2"
            placeholder="Mention ..."
            no-resize
            rows="3"
            v-model="orphanOtherHealthIssues"
          ></b-textarea
        ></b-form-group>
      </b-col>
    </b-form-row>

    <b-form-row align-h="start">
      <!-- next button proceeds to the next
          section of the form after performing validation -->
      <b-button
        align-h="start"
        variant="primary"
        type="submit"
        class="ml-1"
        @click.prevent="handleOrphanHealthSubmit"
        >Next</b-button
      >
    </b-form-row>
  </b-form>
</template>

<script>
import { mapActions } from "vuex";

export default {
  data() {
    return {
      orphanPsychologicalHealth: "",
      orphanPsychologicalHealthState: null,
      orphanOtherHealthIssues: "",

      tmpOrphanPhysicalHealthRadio: "",
      tmpOrphanDisability: "",
      orphanPhysicalHealthState: null,
      tmpOrphanDisabilityState: null
      //   orphanPhysicalHealth: computed prop, doesn't exist in data(){}, // TODO:DONE format the disabled as "disabled:disability"
    };
  },
  computed: {
    orphanPhysicalHealth: function() {
      if (this.tmpOrphanPhysicalHealthRadio === "Disabled") {
        return `Disabled: ${this.tmpOrphanDisability}`;
      } else {
        return "Healthy";
      }
    }
  },
  methods: {
    ...mapActions(["setInvalidHealthForm"]),
    // methods that are named check____[somepropertyname]____Validity use two
    // types of validation i.e., (1) vuelidate for text based inputs & date selectors
    // (2) manual validation for selects, radios and checkboxes
    // both use the ["state": boolean] prop provided by bootstrap-vue
    // true: green bordered input with checkmark indicating input validity
    // false: red bordered input with exclamationmark indicating input invalidity
    checkOrphanPsychologicalHealthValidity: function() {
      this.orphanPsychologicalHealthState =
        this.orphanPsychologicalHealth === "Sociable" ||
        this.orphanPsychologicalHealth === "Not Sociable";
      return this.orphanPsychologicalHealthState;
    },
    checkOrphanPhysicalHealthValidity() {
      if (this.tmpOrphanPhysicalHealthRadio === "Healthy") {
        // if healthy radio is selected or re-selected
        this.orphanPhysicalHealthState = true;
        this.tmpOrphanDisabilityState = null;
        this.tmpOrphanDisability = "";
      } else if (
        this.tmpOrphanPhysicalHealthRadio === "Disabled" &&
        this.tmpOrphanDisability === ""
      ) {
        // if disabled radio is selected without the disability being mentioned
        this.tmpOrphanDisabilityState = false;
        this.orphanPhysicalHealthState = null;
      } else if (this.tmpOrphanPhysicalHealthRadio === "") {
        // if neither is selected
        this.orphanPhysicalHealthState = false;
      } else {
        // if disabled radio is selected and the disability is mentioned
        this.orphanPhysicalHealthState = true;
        this.tmpOrphanDisabilityState = true;
      }
      return this.orphanPhysicalHealthState;
    },
    handleOrphanHealthSubmit: function() {
      // validation check is done one after the other to guide the user on what to fix first
      if (
        this.checkOrphanPhysicalHealthValidity() &&
        this.checkOrphanPsychologicalHealthValidity()
      ) {
        // Set global form validity
        this.setInvalidHealthForm(false);

        // emit `health-info-complete` event to send health info to addOrphan.vue
        this.$emit("health-info-complete", {
          orphanPhysicalHealth: this.orphanPhysicalHealth,
          orphanPsychologicalHealth: this.orphanPsychologicalHealth,
          orphanOtherHealthIssues: this.orphanOtherHealthIssues
        });

        // porceed to the next section
        this.$root.$emit("bv::toggle::collapse", "accordion-3");
      } else this.setInvalidHealthForm(true);
    }
  }
};
</script>

<style></style>
