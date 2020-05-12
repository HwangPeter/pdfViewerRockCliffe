
<template>
  <div class="PDFViewer">
    <v-layout row class="ma-0 pa-3">
      <h2 class="darken-3 orange--text">PDFViewer</h2>
    </v-layout>

    <template>
      <keep-alive>
        <div>
          <v-dialog v-model="dialog" fullscreen class="ma-0 pa-0">
            <template v-slot:activator="{ on }">
              <v-card class="white flex-column d-flex">
                <v-app-bar
                  flat
                  style="background-color: white; border-bottom: 2px solid #fd8c6e"
                  class="header"
                >
                  <v-btn
                    left
                    text
                    small
                    style="background-color: white"
                    v-if="headerName !== 'Home'"
                    @click="handleBackClick()"
                  >
                    <v-icon>arrow_back</v-icon>
                  </v-btn>
                  <span class="title">{{headerName}}</span>
                </v-app-bar>
                <v-layout row wrap class="ma-0">
                  <v-flex xs12>
                    <v-card
                      hover
                      flat
                      class="pa-3 mt-1"
                      v-for="name in sortedPaths"
                      v-on="typeof currentPath[name] !== 'object' ? on : null "
                      v-bind:key="name"
                      @click="handleCardClick(name)"
                    >
                      <div class="d-inline-flex">
                        <v-icon
                          left
                        >{{typeof currentPath[name] == 'object' ? 'folder_open' : 'description'}}</v-icon>
                        <span
                          class="option"
                          v-bind:class="{ isFolder: typeof currentPath[name] == 'object' }"
                          v-if="name.length<20"
                        >{{name}}</span>
                        <span
                          class="option"
                          v-bind:class="{ isFolder: typeof currentPath[name] == 'object' }"
                          v-else
                          style="overflow: hidden;
                    text-overflow: ellipsis;
                    white-space: nowrap;"
                        >{{name}}</span>
                      </div>
                    </v-card>
                  </v-flex>
                </v-layout>
              </v-card>
            </template>

            <v-card>
              <v-app-bar
                flat
                fixed
                class="headline"
                style="border-bottom: 2px solid #EF6C00"
                primary-title
              >
                <!-- Changing current page for workaround to close dialog properly. Forces re-render? -->
                <v-btn
                  text
                  style="background-color: transparent"
                  @click="dialog = false; currentPage++; currentPage = 1;"
                >
                  <v-divider />
                  <v-icon
                    large
                    class="darken-3 orange--text"
                    style="background-color: transparent;"
                  >clear</v-icon>
                </v-btn>
                <span
                  style="overflow: hidden;
                    text-overflow: ellipsis;
                    white-space: nowrap;
                    font-size: 1.25rem;
                      line-height: 1.5;"
                >{{pdfName}}</span>
                <v-spacer></v-spacer>
                <div style="white-space: nowrap">
                  <v-btn text style="background-color: transparent" @click="prevPageClick()">
                    <v-icon
                      large
                      class="darken-3 orange--text"
                      style="background-color: transparent"
                    >arrow_back</v-icon>
                  </v-btn>
                  <span
                    style="font-size: 1.25rem;
                      line-height: 1.5;"
                  >{{this.currentPage}} of {{this.pdfFile.numPages}}</span>
                  <v-btn text style="background-color: transparent" @click="nextPageClick()">
                    <v-icon
                      large
                      class="darken-3 orange--text"
                      style="background-color: transparent"
                    >arrow_forward</v-icon>
                  </v-btn>
                </div>
              </v-app-bar>

              <canvas id="pdfCanvas"></canvas>
            </v-card>
          </v-dialog>
        </div>
      </keep-alive>
    </template>
  </div>
</template>

<script>
// import PDFPopup from "./PDFPopup.vue";
import pdfjsLib from "pdfjs-dist";

export default {
  name: "PDFViewer",

  data: () => ({
    filePaths: {},
    currentPath: {},
    currentPathName: [],
    sortedPaths: [],
    headerName: "Home",
    prevPath: [],
    pdfName: "",
    currentPage: 1,
    pdfFile: "",
    dialog: false,
  }),

  methods: {
    handleCardClick(item) {
      if (item.includes(".pdf")) {
        // PDF clicked, open PDF.
        this.pdfName = item;
        let reqPath = "";
        for (const index in this.currentPathName) {
          reqPath += "/" + this.currentPathName[index];
        }
        reqPath += "/";
        var url = "https://node-api-rockcliffe.herokuapp.com?pdf=" + reqPath + item;

        var loadingTask = pdfjsLib.getDocument(url);
        loadingTask.promise.then(pdf => {
          this.pdfFile = pdf;
          this.renderPDF();
        });
      } else {
        // Folder clicked, open folder.
        this.prevPath.push({
          path: this.currentPath,
          name: this.headerName
        });
        this.currentPathName.push(item);
        this.currentPath = this.currentPath[item];
        this.sortFilesFirst(this.currentPath);
        this.headerName = item;
      }
    },

    isEmpty(obj) {
      for (var x in obj) {
        return false;
      }
      return true;
    },

    handleBackClick() {
      this.currentPathName.pop();
      const previousObj = this.prevPath.pop();
      this.currentPath = previousObj.path;
      this.sortFilesFirst(this.currentPath);
      this.headerName = previousObj.name;
    },

    prevPageClick() {
      if (this.currentPage > 1) {
        this.currentPage--;
        this.renderPDF();
      }
    },

    nextPageClick() {
      if (this.currentPage < this.pdfFile.numPages) {
        this.currentPage++;
        this.renderPDF();
      }
    },

    renderPDF() {
      this.pdfFile.getPage(this.currentPage).then(page => {
        var canvas = document.getElementById("pdfCanvas");
        var context = canvas.getContext("2d");

        var viewport = page.getViewport({ scale: 1 });
        var scale = window.innerWidth / viewport.width;
        var scaledViewport = page.getViewport({ scale: scale });

        canvas.height = scaledViewport.height;
        canvas.width = scaledViewport.width;

        var renderContext = {
          canvasContext: context,
          viewport: scaledViewport
        };
        page.render(renderContext);
      });
    },

    // Sorts obj into array such that folders appear before pdf files.
    sortFilesFirst(data) {
      let fileArr = [];
      this.sortedPaths = [];
      for (const key in data) {
        if (typeof data[key] == "object") {
          this.sortedPaths.push(key);
        } else {
          fileArr.push(key);
        }
      }
      this.sortedPaths = this.sortedPaths.concat(fileArr);
    }
  },

  mounted() {
    if (this.isEmpty(this.filePaths)) {
      fetch("https://node-api-rockcliffe.herokuapp.com")
        .then(response => response.json())
        .then(data => {
          this.filePaths = data;
          this.currentPath = data;
          this.sortFilesFirst(data);
        });
    }
  }
};
</script>

<style scoped>
#pdfCanvas {
  display: block;
  margin-top: 48px;
}
</style>