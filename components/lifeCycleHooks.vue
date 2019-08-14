<template>
  <div>
    <statusPanel />
    {{counter}}
    <button v-on:click="decrement()">-</button>
    <button v-on:click="increment()">+</button>
    <div id="histogram" viewBox="0 0 1000 500">
    </div>
  </div>
</template>

<script>

import * as d3 from 'd3';

import { eventBus } from '~/plugins/eventBus.js';
import statusPanel from '~/components/StatusPanel.vue';
import { setInterval } from 'timers';

// import buttonClick from '~/components/Widgets/buttonClick.vue'

export default {
  name: 'Life-Cycle-Tests',
  components: {
    statusPanel
  },
  data() {
    return {
      MONTHS: this.months,
      // months: ['january', 'february', 'march', 'april', 'may', 'june', 'july', 'august', 'september', 'october', 'november', 'december'],
      SVG: this.svg,
      HEIGHT: this.height,
      WIDTH: this.width,
      MARGIN: this.margin,
      DUMMY: this.dummy,
      WEEKDATA: this.weekData,
      counter: 0,
      // index: this.counter
    }
  },
  methods: {
    increment() {
      this.counter++;
      console.log("increment pressed");
    },
    decrement() {
      this.counter--;
      console.log("decrement pressed");
    },
    drawGraph(WEEKDATA, WIDTH, MARGIN, HEIGHT, SVG, MONTHS) {
      var width = WIDTH;
      var margin = MARGIN;
      var height = HEIGHT;
      var svg = SVG;
      var months = MONTHS;

      console.log("Check WEEKDATA data: ");
      console.log(WEEKDATA);
      var dummy = WEEKDATA[this.counter]
      console.log(WEEKDATA[this.counter]);
      console.log(dummy);

        // create scales
        const xScale = d3.scaleBand()
                        .rangeRound([0, (width - margin.right)/2])
                        .padding(0.1);

        const xAxis = d3.axisBottom(xScale)
                        .tickPadding(10)
                        .tickSizeInner(0)
                        .tickSizeOuter(0)
                        .tickFormat(d => {
                          const dd = new Date(d);
                          return (months[dd.getUTCMonth()]).substr(0, 3) + ' ' + dd.getUTCDate();
                        });

        const yScale = d3.scaleLinear().range([height, 0]);
        const yAxis = d3.axisLeft(yScale)
                        .ticks(5)
                        .tickFormat(d3.format('~s'))
                        .tickPadding(10)
                        .tickSizeOuter(0)
                        .tickSizeInner((width - margin.right) * -1);
        // console.log("weekData[index]");

        // define X & Y domains
        const xDomain = dummy.map(d => d.date.toLocaleString());
        const yDomain = [0, d3.max(dummy, d => {
          return Math.max(d.numberOfTransactions * 1.3, 1000);
        })];

        xScale.domain(xDomain);
        yScale.domain(yDomain).nice();

        svg.append('g')
          .attr('class', 'x axis')
          .attr('transform', 'translate(0,' + height + ')')
          .call(xAxis);

        svg.append('path')
          .attr('class', 'x top-line')
          .attr('transform', 'translate(0,0)')
          .attr('d', 'M0,0H' + (width - margin.right) + 'V0');

        svg.append('path')
          .attr('class', 'x top-line')
          .attr('transform', 'translate(0,0)')
          .attr('d', 'M' + (width - margin.right) + ',0V' + (height));

        svg.append('g')
          .attr('class', 'y axis')
          .call(yAxis)
          .append('text')
          .text('Daily Web Requests')
          .classed('yAxisTitle', true)
          .attr('transform', 'translate(-' + (margin.left - 20) + ', ' + height / 2 + ')rotate(-90)');

        const barsContainer = svg.append('g').attr('class', 'bars');
        const bars = barsContainer.selectAll('.bar').data(dummy).enter().append('g');

        bars.append('rect')
          .attr('class', 'bar')
          .attr('x', d => xScale(d.date.toLocaleString()))
          .attr('width', xScale.bandwidth())
          .attr('y', d => yScale(d.numberOfTransactions))
          .attr('height', d => height - yScale(d.numberOfTransactions));
        bars.append('text')
          .attr('class', 'bar-text')
          .text(d => d.numberOfTransactions > 0 ? d3.format(',.0f')(d.numberOfTransactions) : '')
          .attr('x', d => xScale(d.date.toLocaleString()) + (xScale.bandwidth()) / 2)
          .attr('y', d => yScale(d.numberOfTransactions) - 5)
          .style('text-anchor', 'middle');
    }
  },
  beforeCreate() {
    // Data inactive, events not set up
    console.log('BeforeCreate(): Nothing gets called before this')
  },
  computed: {
    propertyComputed() {
      // console.log('I change when this.property changes.')
      // return this.property
    }
  },
  created() {
    // console.log(this.property);
    // this.property = 'Example property update.'
    console.log('PropertyComputed will update, as this.property is now reactive.')
    // console.log(this.property);
  },
  beforeMount() {
    console.log("Before Mount");
  },
  mounted() {
    var currentDateTime = Date;
    const element = document.getElementById('histogram');
    const margin = { top: 40, bottom: 30, left: 125, right: 1 };
    const width = element.parentElement.offsetWidth - margin.left - margin.right;
    const height = 375 - margin.top - margin.bottom;
    const months = ['january', 'february', 'march', 'april', 'may', 'june', 'july', 'august', 'september',
                  'october', 'november', 'december'];

    d3.select(element).select('svg').remove();
    const svg = d3.select(element).append('svg')
                .attr('width', element.offsetWidth)
                .attr('height', height + margin.top + margin.bottom)
                .append('g')
                .attr('transform', `translate(${margin.left}, ${margin.top})`);

    // console.log("Where am I");      
    // console.log(svg);

    const numberOfDaysWeek = 7;
    const numberOfDaysMon = 30;
    var weekData = [];

    eventBus.$on('jsonData', (json) => {
      
      var data = json["transactionJSON"];
      var parseTime = d3.timeParse("%Y-%m-%dT%H:%M:%SZ");

      // map over data and convert dates to date type
      data = data.map((data) => {
        return {
          date: parseTime(data.date),
          numberOfTransactions: data.numberOfTransactions,
        };
      })

      // Week list of lists 
      for (var i=0;i<data.length;i++) {
        var weekList = parseInt(i/7);
        var dayNumber = i%7;
        if(weekData[weekList] == undefined) {
          weekData[weekList] = [];
        }
        weekData[weekList][dayNumber] = data[i];
      }
      console.log("Here");
      console.log(weekData);
      
      // var dummy = [];
      this.WEEKDATA = weekData,
      this.MONTHS = months;
      this.SVG = svg;
      this.HEIGHT = height;
      this.WIDTH = width;
      this.MARGIN = margin;

      this.drawGraph(this.WEEKDATA, this.WIDTH, this.MARGIN, this.HEIGHT, this.SVG, this.MONTHS);
    });
  },
  //Loads up to here on refresh.
  beforeUpdate() {
    console.log("Before Update");
    console.log("Counter " + this.counter);
    d3.selectAll("g > *").remove();
    // this.drawGraph(this.WEEKDATA, this.WIDTH, this.MARGIN, this.HEIGHT, this.SVG, this.MONTHS);
  },
  updated() {
    //  === this.counter
    // console.log(this.$refs['dom-element'].textContent);
    // console.log(this.counter);
    console.log("Updated");    
    this.drawGraph(this.WEEKDATA, this.WIDTH, this.MARGIN, this.HEIGHT, this.SVG, this.MONTHS);

  },
  beforeDestroy() {
    console.log("Before Destroy");
    delete this.drawGraph(this.WEEKDATA, this.WIDTH, this.MARGIN, this.HEIGHT, this.SVG, this.MONTHS);
  },
  destroyed() {
    console.log("Destroyed");
  }
}
</script>

<style>
</style>