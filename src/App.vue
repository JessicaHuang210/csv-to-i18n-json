<template>
  <h1 class="kjo">ddd</h1>
  <input type="file" accept="text/csv" @change="handleChooseFile" />
  <select v-model="states.lang">
    <option v-for="i in langs" :key="i.id" :value="i.id">{{ i.name }}</option>
  </select>
  <button @click="handleParseToJson">匯出</button>
  <table>
    <thead>
      <tr>
        <th v-for="i in states.csvHead" :key="i">{{ i }}</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="i in states.csvBody" :key="i">
        <td v-for="j in i" :key="j">{{ j }}</td>
      </tr>
    </tbody>
  </table>
</template>

<script>
import { reactive, computed } from "vue";
import Papa from "papaparse";
export default {
  setup() {
    const states = reactive({
      csvHead: [],
      csvBody: [],
      lang: 1
    });
    const langs = computed(() =>
      states.csvHead
        .map((i, index) => {
          return { id: index, name: i };
        })
        .filter((i, index) => index != 0)
    );
    function handleChooseFile(e) {
      const file = e.target.files[0];
      if (file?.type !== "text/csv") return;
      Papa.parse(file, {
        skipEmptyLines: true,
        complete: function(results) {
          states.csvHead = [...results?.data[0]];
          results?.data.shift();
          states.csvBody = results?.data;
        }
      });
    }
    function handleParseToJson() {
      let res = {};
      states.csvBody.forEach(i => {
        res = Object.assign(res, { [i[0]]: i[states.lang] });
      });
      download(
        `${langs.value.find(i => i.id === states.lang).name}.json`,
        JSON.stringify(res)
      );
    }
    function download(filename, text) {
      var element = document.createElement("a");
      element.setAttribute(
        "href",
        "data:application/json;charset=utf-8," + encodeURIComponent(text)
      );
      element.setAttribute("download", filename);
      element.style.display = "none";
      document.body.appendChild(element);
      element.click();
      document.body.removeChild(element);
    }
    return { states, langs, handleChooseFile, handleParseToJson };
  }
};
</script>
<style lang="scss" scoped>
tr th,
tr td {
  border: 1px solid #ccc;
  padding: 10px 15px;
}
tr th {
  background-color: #f7f7f7;
}
</style>
