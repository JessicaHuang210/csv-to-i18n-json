<template>
  <Header />
  <div class="container">
    <div class="section1">
      <transition appear name="fadeDown">
        <div class="section1--header">
          <h1>CSV to JSON</h1>
          <p>上傳指定格式csv檔，輕鬆轉換為i18n可用格式的json檔</p>
        </div>
      </transition>
      <transition appear name="fade">
        <div>
          <label class="section1--upload">
            上傳檔案
            <input type="file" accept="text/csv" @change="handleChooseFile" />
          </label>
        </div>
      </transition>
      <span class="section1--filename">{{ states.csvName }}</span>
    </div>

    <div class="section2" v-if="states.csvHead.length > 0">
      <div class="section2--table">
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
      <div class="section2--download">
        <select class="section2--select" v-model="states.lang">
          <option v-for="i in langs" :key="i.id" :value="i.id">
            {{ i.name }}
          </option>
        </select>
        <button class="section2--btn" @click="handleParseToJson">匯出</button>
      </div>
    </div>

    <div class="section3">
      <div class="section3--demo">
        <img src="csv-demo.png" width="100%" />
      </div>
      <div class="section3--code">
        <span class="section3--code--filename">[ en.json ]</span>
        <pre>
{
  "logout" : "Logout",
  "login" : "Login"
}</pre
        >
      </div>
    </div>
  </div>
</template>

<script>
import { reactive, computed } from "vue";
import Papa from "papaparse";
import Header from "@/components/header";
export default {
  components: {
    Header
  },
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
