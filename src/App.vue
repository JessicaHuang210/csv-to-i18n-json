<template>
  <div class="container">
    <h1>csv檔上傳，區分語言轉換為i18n可用json格式。</h1>
    <div class="content">
      <h3>使用規則</h3>
      <ul>
        <li>A1：基本上打什麼都可以，不計入資料顯示。</li>
        <li>B1 ~ B(n)：語言的名稱，同時也是下載json檔案的名稱。</li>
        <li>選擇語言後點選「匯出」</li>
      </ul>
      <img src="csv-demo.png" width="300" />
      <div class="result">
        匯出的結果：<code>en.json</code>
        <br />
        <br />
        <br />
        <pre>
{
  "logout":"Logout",
  "login":"Login"
}
      </pre
        >
      </div>
    </div>
    <div class="form-group">
      <label class="upload">
        上傳檔案
        <input type="file" accept="text/csv" @change="handleChooseFile" />
      </label>
      <span class="filename">{{ states.csvName }}</span>
    </div>
    <div class="form-group" v-if="states.csvHead.length > 0">
      選擇語言：<select v-model="states.lang">
        <option v-for="i in langs" :key="i.id" :value="i.id">{{
          i.name
        }}</option>
      </select>
      <button @click="handleParseToJson">匯出</button>
    </div>
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
  </div>
</template>

<script>
import { reactive, computed } from "vue";
import Papa from "papaparse";
export default {
  setup() {
    const states = reactive({
      csvHead: [],
      csvBody: [],
      csvName: "",
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
      states.csvName = file.name;
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
