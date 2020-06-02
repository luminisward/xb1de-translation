<template>
  <v-card class="result-table">
    <v-container fluid>
      <form @submit.prevent="query">
        <v-row>
          <v-col cols="12" sm="3">
            <v-select v-model="queryLanguage" :items="languages" label="Query Language"></v-select>
          </v-col>
          <v-col cols="12" sm="6">
            <v-select
              v-model="resultLanguages"
              :items="languages.filter(lan=>lan!=queryLanguage)"
              label="Result Language"
              multiple
            ></v-select>
          </v-col>
          <v-col cols="12" sm="3">
            <v-text-field v-model.number="limit" label="Limit" type="number"></v-text-field>
          </v-col>
        </v-row>

        <v-row>
          <v-col cols="12" sm="6">
            <v-combobox v-model="bdats" label="Bdat" multiple clearable chips></v-combobox>
          </v-col>

          <v-col cols="12" sm="6">
            <v-combobox v-model="tables" label="Table" multiple clearable chips></v-combobox>
          </v-col>
        </v-row>

        <v-row>
          <v-col>
            <v-text-field v-model="queryString" label="Search Text" counter outlined></v-text-field>
          </v-col>
        </v-row>

        <v-row>
          <v-col>
            <v-btn color="primary" type="submit">Search</v-btn>
          </v-col>
        </v-row>
        <v-spacer></v-spacer>
      </form>
      <v-data-table :headers="headers" :items="tableDataWithKey" item-key="key" :loading="loading"></v-data-table>
    </v-container>

    <v-snackbar v-model="snackbar" top>
      {{ snackbarText }}
      <v-btn color="blue" text @click="snackbar = false">Close</v-btn>
    </v-snackbar>
  </v-card>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      languages: ["cn", "fr", "gb", "ge", "it", "jp", "kr", "sp", "tw"],
      queryLanguage: "cn",
      queryString: "",
      resultLanguages: [],
      bdats: [],
      tables: [],
      limit: 100,
      search: "",
      tableData: [],
      snackbar: false,
      snackbarText: "",
      loading: false
    };
  },
  computed: {
    headers() {
      return [
        { text: "bdat.table.id", value: "key", sortable: false },
        ...[this.queryLanguage, ...this.resultLanguages].map(language => ({
          text: language,
          value: language,
          sortable: false
        }))
      ];
    },
    tableDataWithKey() {
      return this.tableData.map(row => ({
        key: [row.bdat, row.table, row.id].join("."),
        ...row
      }));
    }
  },
  watch: {
    queryLanguage() {
      this.resultLanguages = this.resultLanguages.filter(
        lan => lan != this.queryLanguage
      );
    },
    limit() {
      if (this.limit > 500) {
        this.$nextTick(() => {
          this.limit = 500;
        });
      } else if (this.limit < 1) {
        this.$nextTick(() => {
          this.limit = 1;
        });
      }
    }
  },
  methods: {
    async query() {
      try {
        this.loading = true;
        const { data } = await axios.post(
          `${process.env.VUE_APP_API}/translations${window.location.search}`,
          {
            query_language: this.queryLanguage,
            query_string: this.queryString,
            result_languages: this.resultLanguages,
            bdats: this.bdats,
            tables: this.tables,
            limit: this.limit
          }
        );

        if (data.total === 0) {
          this.showSnackbar(`No data.`);
          return;
        }

        if (data.limit > 0 && data.total >= data.limit) {
          this.showSnackbar(`Too many results, limited to ${data.limit} rows.`);
        }

        const results = data.results;

        const languages = Object.keys(results);

        const resultsTranspose = [];
        for (const [i, row] of results[this.queryLanguage].entries()) {
          const newRow = {
            bdat: row.bdat,
            table: row.table,
            id: row.row_id
          };
          for (const language of languages) {
            newRow[language] = results[language][i].name;
          }
          resultsTranspose.push(newRow);
        }

        this.tableData = resultsTranspose;
      } catch (error) {
        if (error.response) {
          this.showSnackbar(error.response.data.errmsg);
        } else {
          this.showSnackbar(error.message);
        }
      } finally {
        this.loading = false;
      }
    },
    showSnackbar(text) {
      this.snackbarText = text;
      this.snackbar = true;
    }
  }
};
</script>

<style lang="css" scoped>
.result-table {
  margin: 0.8em;
}
</style>
