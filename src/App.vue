<script>
import { computed, defineComponent, ref } from "vue";
import FilesViewer from "./components/FileViewer";

import fs from "fs";
import pathModule from "path";
import { app } from "@electron/remote";

const formatSize = (size) => {
  const i = Math.floor(Math.log(size) / Math.log(1024));
  return (
    (size / Math.pow(1024, i)).toFixed(2) * 1 +
    " " +
    ["B", "kB", "MB", "GB", "TB"][i]
  );
};

export default defineComponent({
  components: { FilesViewer },
  setup() {
    const path = ref(app.getAppPath());
    const files = computed(() => {
      const fileName = fs.readdirSync(path.value);
      return fileName
        .map((file) => {
          const stats = fs.statSync(pathModule.join(path.value, file));
          return {
            name: file,
            size: stats.isFile() ? formatSize(stats.size ?? 0) : null,
            directory: stats.isDirectory(),
          };
        })
        .sort((a, b) => {
          if (a.directory === b.directory) {
            return a.name.localeCompare(b.name);
          }
          return a.directory ? -1 : 1;
        });
    });

    const back = () => {
      path.value = pathModule.dirname(path.value);
    };

    const open = (folder) => {
      path.value = pathModule.join(path.value, folder);
    };

    const searchString = ref("");
    const filteredFiles = computed(() => {
      return searchString.value
        ? files.value.filter((s) => s.name.startsWith(searchString.value))
        : files.value;
    });
    return { path, files: filteredFiles, back, open, searchString };
  },
});
</script>

<template>
  <div class="container mt-2">
    <h4>{{ path }}</h4>
    <div class="from-group mt-4 mb-2">
      <input
        class="form-control from-control-sm"
        placeholder="File search"
        v-model="searchString"
      />
    </div>
    <FilesViewer :files="files" @back="back" @folderClick="open($event.name)" />
  </div>
</template>