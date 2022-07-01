<template>
  <div>
    <form @submit.prevent="submitHandler">
      <input
        type="file"
        @change="fileUploadHandler"
        accept=".xls,.xlsx, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet,application/vnd.ms-excel"
      />
      <button :disabled="!fileCount">Submit</button>
    </form>
    <div>{{ directConversionRate }}</div>
    <div v-for="(value, key) in conversionData">
      {{ key }}: {{ value[key] }}
      <div v-for="(subVal, k) in value[key]">{{ k }}: {{ subVal }}</div>
    </div>
    <div>{{ directSalesRate }}</div>
    <div></div>
  </div>
</template>

<script>
import readXlsxFile from "read-excel-file";

export default {
  name: "Index",
  data: () => ({
    directConversionRate: "",
    rowsWithConversion: [],
    conversionData: {},
    directSalesRate: "",
    fileCount: 0,
  }),
  props: {
    msg: String,
  },
  methods: {
    submitHandler: function () {
      let input = document.querySelector('input[type="file"]');
      readXlsxFile(input.files[0]).then((rows) => {
        let indices = {};
        ["Converted Rental", "Touchpoints"].forEach((i) => {
          indices[i] = rows[0].indexOf(i);
        });
        let headerRow = rows.shift();
        this.rowsWithConversion = rows.filter(
          (row) => row[indices["Converted Rental"]].toLowerCase() === "yes"
        );
        let rowsWithDirectSales = rows.filter(
          (row) => row[indices["Converted Rental"]].toLowerCase() === "no"
        );
        let conversionSources = [
          ...new Set(
            this.rowsWithConversion
              .map((row, i) => {
                let json = JSON.parse(row[indices["Touchpoints"]]);
                for (let j = 0; j < json.length; j++) {
                  let { referrer_medium, referrer_source } = json[j];
                  console.log(i, j, referrer_source, referrer_medium);
                  if (typeof referrer_source !== `undefined`)
                    return referrer_source;
                }
              })
              .filter((source) => source !== undefined)
          ),
        ].sort();
        console.log(`=====`);
        let directSources = [
          ...new Set(
            rowsWithDirectSales
              .map((row, i) => {
                let json = JSON.parse(row[indices["Touchpoints"]]);
                for (let j = 0; j < json.length; j++) {
                  let { referrer_medium, referrer_source } = json[j];
                  console.log(i, j, referrer_source, referrer_medium);
                  if (typeof referrer_source !== `undefined`)
                    return referrer_source;
                }
              })
              .filter((source) => source !== undefined)
          ),
        ].sort();
        console.log(`conversionSources`, conversionSources);
        console.log(`directSources`, directSources);

        let conversionData = {};

        let conversionBreakdown = conversionSources.forEach((source) => {
          console.log(`source: `, source);
          conversionData[source] = [];

          this.rowsWithConversion.forEach((row) => {
            let json = JSON.parse(row[indices["Touchpoints"]]);
            json.forEach((jsonObj) => {
              let { referrer_source, referrer_medium } = jsonObj;
              if (referrer_source === source) {
                typeof conversionData[source][referrer_medium] === `undefined`
                  ? (conversionData[source][referrer_medium] = 1)
                  : conversionData[source][referrer_medium]++;
              }
            });
          });
        });

        this.conversionData = conversionData;

        console.log(conversionData);

        this.directConversionRate = `Conversion Percentage = ${(
          (this.rowsWithConversion.length / rows.length) *
          100
        ).toFixed(2)}%`;
        this.directSalesRate = `Direct Sales Percentage = ${(
          (rowsWithDirectSales.length / rows.length) *
          100
        ).toFixed(2)}%`;
      });
    },
    fileUploadHandler: function (ev) {
      this.fileCount = ev.target.files.length;
    },
  },
};
</script>

<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
