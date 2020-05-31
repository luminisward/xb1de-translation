<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div class="d-flex align-center title">Xenoblade DE Text Search Tool</div>

      <v-spacer></v-spacer>
      <v-btn href="https://xenoblade.cn" target="_blank" text>
        <span class="subtitle-1">蒙纳多研究本部</span>
      </v-btn>
    </v-app-bar>

    <v-content>
      <ResultTable />
    </v-content>
    <v-footer padless>
      <v-col class="text-center" cols="12">
        <strong>
          Powered by
          <a target="_blank" href="https://github.com/Thealexbarney/XbTool">XbTool</a>
        </strong>
      </v-col>
    </v-footer>
  </v-app>
</template>

<script>
import ResultTable from "./components/ResultTable";

export default {
  name: "App",

  components: {
    // HelloWorld,
    ResultTable
  },

  data: () => ({
    //
  })
};
</script>
