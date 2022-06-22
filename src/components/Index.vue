<template>
  <div>
    <form @submit.prevent="submitHandler">
      <input type="file" />
      <button>Submit</button>
    </form>
    <div>{{ directConversionRate }}</div>
    <div>{{ directSalesRate }}</div>
    <div></div>
  </div>
</template>

<script>
import readXlsxFile from "read-excel-file";

export default {
  name: "HelloWorld",
  data: () => ({
    directConversionRate: "",
    directSalesRate: "",
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
        let rowsWithConversion = rows.filter(
          (row) => row[indices["Converted Rental"]].toLowerCase() === "yes"
        );
        let rowsWithDirectSales = rows.filter(
          (row) => row[indices["Converted Rental"]].toLowerCase() === "no"
        );
        let conversionSources = [
          ...new Set(
            rowsWithConversion
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
        this.directConversionRate = `Conversion Percentage = ${(
          (rowsWithConversion.length / rows.length) *
          100
        ).toFixed(2)}%`;
        this.directSalesRate = `Direct Sales Percentage = ${(
          (rowsWithDirectSales.length / rows.length) *
          100
        ).toFixed(2)}%`;
      });
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
