<script src="https://d3js.org/d3.v6.js"></script>
<script src="https://unpkg.com/vue"></script>

<template>
  <v-app>
    <v-container>
      <v-row>
        <v-col id="base_distribution" cols="4" />
        <v-col id="markov_chain" cols="4" />
        <v-col id="markov_hist" cols="4" />
      </v-row>
    </v-container>
  </v-app>
</template>

<script>
import * as d3 from "d3";
import EventBus from '../resources/EventBus';

export default {
  data () {
    return {
      params: {},
    };
  },
  props: {
    formData: Object
  },
  mounted() {
    EventBus.$on('pushFormData', (formData) => {
      this.formData = formData;
      this.params = formData;
      this.params.numBins = parseInt(formData.numBins);
      this.params.numSamples = parseInt(formData.numSamples);
      this.params.mean = parseFloat(formData.mean);
      this.params.std = parseFloat(formData.std);
      this.params.lowerBound = parseFloat(formData.lowerBound);
      this.params.upperBound = parseFloat(formData.upperBound);
      this.generateBaseDistribution();
      var markovIndex = this.generateMarkovChain();
      this.generateMarkovHist(markovIndex);
    });
  },
  methods: {
    probDist(lowerBound, upperBound, numBins, mean, std) {
      var norm_array = this.linspace(lowerBound, upperBound, numBins);
      var pdf_array = [];
      for(var i=0; i < numBins; i++) {
        pdf_array.push(this.norm_pdf(norm_array[i], mean, std));
      };

      return [pdf_array, norm_array];
    },

    getRandInt(min, max) {
      var min = Math.ceil(min);
      var max = Math.floor(max);
      return Math.floor(Math.random() * (max - min + 1) + min);
    },

    metropolisAlgorithm(pdf_array, numBins, numSamples) {
      var startIndex = this.getRandInt(0, numBins - 1);
      var markovIndex = [startIndex];
      var count = 0;

      for(var i=0; i < numSamples; i++) {
        var coin = this.getRandInt(0, 1);
        var rand = Math.random();

        if(coin === 1) {
          if(markovIndex[count] === pdf_array.length - 1) {
            markovIndex.push(markovIndex[count]);
          } 
          else if(pdf_array[markovIndex[count] + 1] > pdf_array[markovIndex[count]]) {
            markovIndex.push(markovIndex[count] + 1);
          }
          else {
            if(rand < (pdf_array[markovIndex[count] + 1] / pdf_array[markovIndex[count]])) {
              markovIndex.push(markovIndex[count] + 1);
            }
            else {
              markovIndex.push(markovIndex[count]);
            }
          }
        }
        else {
          if(markovIndex[count] === 0) {
            markovIndex.push(markovIndex[count]);
          }
          else if(pdf_array[markovIndex[count] - 1] > pdf_array[markovIndex[count]]) {
            markovIndex.push(markovIndex[count] - 1);
          }
          else {
            if(rand < (pdf_array[markovIndex[count] - 1] / pdf_array[markovIndex[count]])) {
              markovIndex.push(markovIndex[count] - 1);
            }
            else {
              markovIndex.push(markovIndex[count]);
            }
          }
        }
        count += 1;
      }
      return markovIndex;
    },

    linspace(lowerBound, upperBound, cardinality) {
      var arr = [];
      var step = (upperBound - lowerBound) / (cardinality - 1);
      for(var i=0; i < cardinality; i++) {
        arr.push(lowerBound + (step * i));
      }
      return arr;
    },

    norm_pdf(point, mean, std) {
      var norm_point = (point - mean) / std;
      var num = Math.exp(-Math.pow(norm_point, 2));
      var den = Math.sqrt(2 * Math.PI);

      return num / den
    },

    generateBaseDistribution() {

      let margin = { top: 10, right: 10, bottom: 50, left: 60 },
      width = 400 - margin.left - margin.right,
      height = 400 - margin.top - margin.bottom;

      var [pdf_array, norm_array] = this.probDist(this.params.lowerBound, this.params.upperBound, this.params.numBins, this.params.mean, this.params.std)

      var base_dist_data = [];
      for(var i=0; i < this.params.numBins; i++) {
        var obj = {};
        obj.key = norm_array[i];
        obj.value = pdf_array[i];
        base_dist_data.push(obj);
      };

      let svg = d3
        .select("#base_distribution")
        .attr("id", "base_distribution")
        .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

      let minX = this.params.lowerBound;
      let maxX = this.params.upperBound;

      let x = d3.scaleLinear().domain([minX - ((this.params.numBins) / (2 * this.params.numBins)), maxX + ((this.params.numBins) / (2 * this.params.numBins))]).range([0, width]);

      svg.append("g")
         .attr("transform", "translate(0," + height + ")")
         .call(d3.axisBottom(x));

      svg.append("text")
         .attr("transform", "translate(" + width / 2 + " ," + (height + margin.top + 30) + ")")
         .style("text-anchor", "middle")
         .text("x-axis Label");

      let y = d3.scaleLinear().domain([0, d3.max(pdf_array)]).range([height, 0]);

      svg.append("g").call(d3.axisLeft(y));

      svg.append("text")
         .attr("transform", "rotate(-90)")
         .attr("y", -40)
         .attr("x", 0 - height / 2)
         .style("text-anchor", "middle")
         .text("y-axis Label");

      svg.selectAll(".bar")
         .data(base_dist_data)
         .enter().append("rect")
         .attr("class", "bar")
         .attr("x", function(d) { return x(d.key) - 20 * (norm_array[1] - norm_array[0]); })
         .attr("y", function(d) { return y(d.value); })
         .attr("width", 40 * (norm_array[1] - norm_array[0]))
         .attr("height", function(d) { return height - y(d.value); })
        //  .style("fill", "blue")
        ;
    },

    generateMarkovChain() {

      var [pdf_array, norm_array] = this.probDist(this.params.lowerBound, this.params.upperBound, this.params.numBins, this.params.mean, this.params.std)
      var markovIndex = this.metropolisAlgorithm(pdf_array, this.params.numBins, this.params.numSamples)

      var markov_chain_data = [];
      for(var i=0; i < this.params.numSamples; i++) {
        var obj = {};
        obj.key = markovIndex[i];
        obj.value = i;
        markov_chain_data.push(obj);
      };

      let margin = { top: 10, right: 10, bottom: 50, left: 60 },
      width = 400 - margin.left - margin.right,
      height = 400 - margin.top - margin.bottom;

      let svg = d3
        .select("#markov_chain")
        .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

      let x = d3.scaleLinear().domain([0, this.params.numBins]).range([0, width]);

      svg.append("g")
         .attr("transform", "translate(0," + height + ")")
         .call(d3.axisBottom(x));

      svg.append("text")
         .attr("transform", "translate(" + width / 2 + " ," + (height + margin.top + 30) + ")")
         .style("text-anchor", "middle")
         .text("x-axis Label");

      let y = d3.scaleLinear().domain([0, this.params.numSamples]).range([height, 0]);

      svg.append("g").call(d3.axisLeft(y));

      svg.append("text")
         .attr("transform", "rotate(-90)")
         .attr("y", -40)
         .attr("x", 0 - height / 2)
         .style("text-anchor", "middle")
         .text("y-axis Label");

      let circles = svg.selectAll("circle")
                       .data(markov_chain_data, function(d) {
                         return d["value"];
                       });

      circles.enter()
             .append("circle")
             .attr("cy", function (d) {
               return y(d["value"]);
             })
             .attr("cx", function (d) {
               return x(d["key"]);
             })
             .attr("r", 5)
             // .style("fill", "blue");

      return markovIndex;
    },

    generateMarkovHist(markovIndex) {
      const fq = {};

      for (const num of markovIndex) {
        fq[num] = fq[num] ? fq[num] + 1 : 1;
      }

      var values = [];
      for (const value in fq) {
        values.push(fq[value]);
      }

      var keys = [];
      for (const key in fq) {
        keys.push(parseFloat(key));
      }

      console.log(keys);
      console.log(values);

      var maxFreq = Math.max(...values);

      const bins = [];

      for(var i = 0; i < this.params.numBins; i++) {
        bins.push(i);
      }

      var markov_hist_data = [];
      for(var i = 0; i < keys.length; i++) {
        var obj = {};
        obj.key = keys[i];
        obj.value = values[i];
        markov_hist_data.push(obj);
      };
      console.log(markov_hist_data);

      var [pdf_array, norm_array] = this.probDist(this.params.lowerBound, this.params.upperBound, this.params.numBins, this.params.mean, this.params.std)

      let margin = { top: 10, right: 10, bottom: 50, left: 60 },
      width = 400 - margin.left - margin.right,
      height = 400 - margin.top - margin.bottom;

      let svg = d3
        .select("#markov_hist")
        .attr("id", "markov_hist")
        .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

      let minX = bins[0];
      let maxX = bins[this.params.numBins - 1];

      let x = d3.scaleLinear().domain([minX - ((this.params.numBins) / (2 * this.params.numBins)), maxX + ((this.params.numBins) / (2 * this.params.numBins))]).range([0, width]);

      svg.append("g")
         .attr("transform", "translate(0," + height + ")")
         .call(d3.axisBottom(x));

      svg.append("text")
         .attr("transform", "translate(" + width / 2 + " ," + (height + margin.top + 30) + ")")
         .style("text-anchor", "middle")
         .text("x-axis Label");

      let y = d3.scaleLinear().domain([0, maxFreq]).range([height, 0]);

      svg.append("g").call(d3.axisLeft(y));

      svg.append("text")
         .attr("transform", "rotate(-90)")
         .attr("y", -40)
         .attr("x", 0 - height / 2)
         .style("text-anchor", "middle")
         .text("y-axis Label");

      svg.selectAll(".bar")
         .data(markov_hist_data)
         .enter().append("rect")
         .attr("class", "bar")
         .attr("x", function(d) { return x(d.key) - 20 * (norm_array[1] - norm_array[0]); })
         .attr("y", function(d) { return y(d.value); })
         .attr("width", 2 * (norm_array[this.params.numBins - 1] - norm_array[0]))
         .attr("height", function(d) { return height - y(d.value); })
         // .style("fill", "blue")
        ;
    }
  }
};
</script>

<style scoped>
  .bar {
    fill: blue,
    
  }
</style>