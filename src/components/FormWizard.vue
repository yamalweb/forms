<template>
  <div>
    <div v-if="wizzardInProgress" v-show="asyncState !== 'pending'">
      <keep-alive>
        <Component
          ref="currentStep"
          :is="currentStep"
          :wizardData="form"
          @updateAsyncState="updateAsyncState"
        />
      </keep-alive>

      <div class="progress-bar">
        <div :style="`width: ${progress}%;`"></div>
      </div>

      <!-- Actions -->
      <div class="buttons">
        <button @click="goBack" v-if="currentStepNumber > 1" class="btn-outlined">
          Back
        </button>
        <button @click="nextButtonActions" class="btn">
          {{ isLastStep ? "Confirm" : "Next" }}
        </button>
      </div>

      <pre><code>{{ form }}</code></pre>
    </div>
    <div v-else>
      <h1 class="title">343234 234234</h1>
      <h2 class="subtitle">Awesome!</h2>
    </div>
    <div class="loading-wraper" v-if="asyncState === 'pending'">
      <div class="loader">
        <img src="/spinner.svg" alt="" />
        <p>
          ipsum Lorem ipsum, dolor sit amet consectetur adipisicing elit. Molestias
          delectus tempora totam praesentium nesciunt. Deleniti deserunt facilis commodi
          architecto perspiciatis nobis culpa enim voluptas possimus, harum itaque? Animi,
          voluptates neque.
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import { postFormToDB } from "../api/";
import FormPlanPicker from "./FormPlanPicker";
import FormUserDetails from "./FormUserDetails";
import FormAddress from "./FormAddress";
import FormReviewOrder from "./FormReviewOrder";
export default {
  name: "FormWizard",
  components: {
    FormPlanPicker,
    FormUserDetails,
    FormAddress,
    FormReviewOrder,
  },
  data() {
    return {
      currentStepNumber: 1,
      form: {
        plan: null,
        email: null,
        name: null,
        password: null,
        address: null,
        recipient: null,
        chocolate: false,
        otherTreat: false,
      },
      steps: ["FormPlanPicker", "FormUserDetails", "FormAddress", "FormReviewOrder"],
      asyncState: null,
    };
  },
  computed: {
    isLastStep() {
      return this.currentStepNumber === this.length;
    },
    wizzardInProgress() {
      return this.currentStepNumber <= this.length;
    },
    length() {
      return this.steps.length;
    },
    currentStep() {
      return this.steps[this.currentStepNumber - 1];
    },
    progress() {
      return (this.currentStepNumber / this.length) * 100;
    },
  },
  methods: {
    updateAsyncState(state) {
      this.asyncState = state;
    },
    nextButtonActions() {
      this.$refs.currentStep
        .submit()
        .then((stepData) => {
          Object.assign(this.form, stepData);
          if (this.isLastStep) {
            this.submitOrder();
          } else {
            this.goNext();
          }
        })
        .catch((error) => console.log(error));
    },
    submitOrder() {
      this.asyncState = "pending";
      postFormToDB(this.form).then(() => {
        console.log("form submited!", this.form);
        this.asyncState = "success";
        this.currentStepNumber++;
      });
    },

    goBack() {
      this.currentStepNumber--;
    },
    goNext() {
      this.currentStepNumber++;
    },
  },
};
</script>
