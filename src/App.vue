<template>
  <button @click="getme(wavemaker.projects)">Analyse Data Project</button>

  <button
    v-for="(proj, index) in displayOutput"
    :key="index"
    @click="cycleProject(index)"
  >
    {{ proj.title }}
  </button>
</template>

<script>
import wavemaker from "./sampledata.json";
import { uuid } from "vue-uuid";
import { db } from "./db.js";
import marked from "marked";
export default {
  name: "App",
  data() {
    return {
      db,
      uuid: uuid.v4(),
      wavemaker,
      displayOutput: [],
      filearray: [],
      settings: {},
    };
  },
  methods: {
    getme(data) {
      if (!data) {
        data = this.wavemaker;
      }
      Object.keys(data).forEach((k) => {
        this.displayOutput.push({ key: k, title: data[k].title });
      });
    },

    cycleProject(k) {
      this.settings = {
        settings: {
          ProjectName: this.wavemaker.projects[k].title,
        },
        uuid: this.uuid,
      };

      this.db.Settings.add(this.settings, this.settings.uuid);

      let proj = this.wavemaker.projects[k].data;
      // get the writer elements and pages and deal with then
      let writerid = this.uuid;
      this.filearray = [];
      this.loopfiles(proj.writer, this.filearray, writerid);

      let o = {};
      o.title = "";
      o.description = "";
      o.files = this.filearray;
      o.uuid = writerid;

      this.db.Writer.add(o, o.uuid);
      // END of the Writer Code
    },
    loopfiles(arr, parent, writerid) {
      // this does all the work for the files creation etc and building the array
      arr.forEach((f) => {
        let file = {
          uuid: this.uuid,
          type: "file",
          children: [],
          open: true,
        };
        // create and add the FILE
        let o = {};
        o.uuid = file.uuid;
        o.writerid = writerid;
        o.contents = marked(f.content); /// needs to be converted to html here
        o.title = f.title;
        o.notes = [];

        if (f.notes.length) {
          f.notes.forEach((card) => {
            let cardobj = {};
            cardobj.uuid = this.uuid;
            cardobj.title = card.title;
            cardobj.description = card.content;
            cardobj.showdesc = true;
            cardobj.content = "";
            cardobj.labels = [];
            cardobj.images = [];
            cardobj.style = "";
            cardobj.options = {};
            cardobj.color = "--card" + card.backgroundColor;
            o.notes.push({ uuid: cardobj.uuid });
            this.db.Cards.add(cardobj, cardobj.uuid);
          });
        }

        this.db.Files.add(o, o.uuid);

        // we are going tocreate and link a card for all notes

        if (arr.children.length) {
          this.loopfiles(f.children, file.children, writerid);
        }
        parent.push(file); // add the file to the file structure
      });
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;

  color: #2c3e50;
}
</style>
