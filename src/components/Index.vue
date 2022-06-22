<template>
  <div>
    <form @submit.prevent="submitHandler">
      <input type="file" />
      <button>Submit</button>
    </form>
    <div>{{ directConversionRate }}</div>
    <div></div>
    <div></div>
    <div></div>
  </div>
</template>

<script>
import readXlsxFile from "read-excel-file";

export default {
  name: "HelloWorld",
  data: () => ({
    directConversionRate: "",
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
        let headerRow = rows.shift(),
          conversions = 0;
        let rowsWithConversion = rows.filter(
          (row) => row[indices["Converted Rental"]].toLowerCase() === "yes"
        );
        let rowsDirectSales = rows.filter(
          (row) => row[indices["Converted Rental"]].toLowerCase() === "no"
        );
        // rows.forEach((row) => {
        //   if (row[indices["Converted Rental"]].toLowerCase() === "yes")
        //     conversions++;
        // });
        let conversionSources = [
          ...new Set(
            rowsWithConversion.map((row, i) => {
              let json = JSON.parse(row[indices["Touchpoints"]]);
              for (let j = 0; j < json.length; j++)
                console.log(
                  i,
                  j,
                  json[j].referrer_source,
                  json[j].referrer_medium
                );
            })
          ),
        ].sort();
        console.log(conversionSources);
        this.directConversionRate = `Conversion Percentage = ${(
          (rowsWithConversion.length / rows.length) *
          100
        ).toFixed(2)}%`;
        // rows.forEach((row) => {
        //   let json = JSON.parse(row[indices["Touchpoints"]]);
        //   console.log(json);
        // });
      });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
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
