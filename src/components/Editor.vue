<template>
  <div>
    <p class="text-capitalize">{{ config.label }}</p>
    <div :id="uid" class="editor-wrapper" :style="config.style || {}" />
  </div>
</template>
<script>
import "jsoneditor/dist/jsoneditor.min.css";
const JsonEditor = require("jsoneditor");

function uuidv4HTML() {
  return (
    "i" +
    ([1e7] + -1e3 + -4e3 + -8e3 + -1e11).replace(/[018]/g, (c) =>
      (
        c ^
        (crypto.getRandomValues(new Uint8Array(1))[0] & (15 >> (c / 4)))
      ).toString(16)
    )
  );
}

const editorFactory = (uid, config) =>
  new JsonEditor(document.querySelector(`#${uid}`), config);

export default {
  props: ["config"],
  data: () => ({
    uid: uuidv4HTML(),
    editor: {},
  }),

  mounted() {
    this.editor = editorFactory(
      this.uid,
      this.config.editorConfig || { mode: "code" }
    );
  },

  methods: {
    set(v) {
      this.editor.set(v);
    },
  },
};
</script>
<style>
.editor-wrapper .jsoneditor-menu,
.editor-wrapper .jsoneditor-statusbar {
  display: none;
}
</style>
