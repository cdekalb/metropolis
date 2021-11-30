<template>
  <div class="param-input">
    <h1>Enter Parameters</h1>
    <b-form @submit="handleSubmit">

      <b-form-group id="input-group-1" label="Distribution Type:" label-for="input-1">
        <b-form-select
          id="input-1"
          v-model="distType"
          :options="['Normal']"
          required
        ></b-form-select>
      </b-form-group>

      <b-form-group id="input-group-2" label="Number of Bins:" label-for="input-2">
        <b-form-input
          id="input-2"
          type="number"
          v-model="numBins"
          placeholder="Enter number of bins"
          required
        ></b-form-input>
      </b-form-group>

      <b-form-group id="input-group-3" label="Number of Samples:" label-for="input-3">
        <b-form-input
          id="input-3"
          type="number"
          v-model="numSamples"
          placeholder="Enter number of samples"
          required
        ></b-form-input>
      </b-form-group>

      <b-form-group id="input-group-4" label="Mean:" label-for="input-4">
        <b-form-input
          id="input-4"
          type="number"
          v-model="mean"
          placeholder="Enter mean"
          required
        ></b-form-input>
      </b-form-group>

      <b-form-group id="input-group-5" label="Standard Deviation:" label-for="input-5">
        <b-form-input
          id="input-5"
          type="number"
          v-model="std"
          placeholder="Enter standard deviation"
          required
        ></b-form-input>
      </b-form-group>

      <b-form-group id="input-group-6" label="Lower Bound:" label-for="input-6">
        <b-form-input
          id="input-6"
          type="number"
          v-model="lowerBound"
          placeholder="Enter lower bound"
          required
        ></b-form-input>
      </b-form-group>

      <b-form-group id="input-group-7" label="Upper Bound:" label-for="input-7">
        <b-form-input
          id="input-7"
          type="number"
          v-model="upperBound"
          placeholder="Enter upper bound"
          required
        ></b-form-input>
      </b-form-group>
      <br/>

      <div class="buttons">
        <b-button type="button" variant="primary" @click="handleSubmit">Submit</b-button>
        <b-button type="reset" variant="danger">Reset</b-button>
      </div>
    </b-form>
    <b-card class="mt-3" header="Form Data Result">
      <pre class="m-0">{{ formData }}</pre>
    </b-card>

    <graph-view :formData=formData />
  </div>

  
</template>

<script>
import GraphView from './GraphView.vue';
import EventBus from '../resources/EventBus';

export default {
  components: { GraphView },
  name: 'param-input',
  data() {
    return {
      formData: {},
      distType: '',
      numBins: '',
      numSamples: '',
      mean: '',
      std: '',
      lowerBound: '',
      upperBound: ''
    }
  },
  methods: {
    handleSubmit() {
      this.formData = {
        distType: this.distType,
        numBins: this.numBins,
        numSamples: this.numSamples,
        mean: this.mean,
        std: this.std,
        lowerBound: this.lowerBound,
        upperBound: this.upperBound
      }
      EventBus.$emit('pushFormData', this.formData);
    }
  }
}
</script>

<style scoped>
  div.form-group, h1 {
    max-width: 350px;
    margin: auto;
  }

  div.buttons {
    max-width: 350px;
    margin: auto;
  }

  .mt-3 {
    text-align: center;
    margin: auto;
  }
</style>
