<template>
  <v-app>
    <v-container fluid>
      <v-row>
        <v-col>
          <v-card flat>
            <v-card-title>REQUEST</v-card-title>
            <v-card-text>
              <v-text-field
                label="https://acme.com"
                filled
                v-model="url"
              ></v-text-field>
              <div>
                <v-btn
                  :depressed="selectedMethod !== method"
                  :style="{
                    transform: selectedMethod === method ? 'scale(1.15)' : null,
                  }"
                  :color="color"
                  dark
                  width="96px"
                  v-for="({ method, color }, i) of methods"
                  :key="i"
                  @click="() => (selectedMethod = method)"
                  class="mr-3"
                  >{{ method }}</v-btn
                >
              </div>

              <editor
                :config="editors.headers"
                class="pt-3"
                v-model="headers"
              />
              <editor :config="editors.query" class="pt-3" v-model="query" />
              <editor
                :config="editors.body"
                class="pt-3"
                v-model="body"
                v-if="selectedMethod !== 'head' && selectedMethod !== 'get'"
              />
              <div class="d-flex justify-end pt-3">
                <v-btn
                  depressed
                  color="success"
                  width="96px"
                  @click="ajax"
                  :disabled="!isOk"
                  :loading="loading"
                  >Send</v-btn
                >
              </div>
            </v-card-text>
          </v-card>
        </v-col>
        <v-col>
          <v-card flat>
            <v-card-title>RESPONSE</v-card-title>
            <v-card-text>
              <editor :config="editors.responseHeaders" ref="responseHeaders" />
              <editor
                :config="editors.responseBody"
                class="pt-3"
                ref="responseBody"
              />
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>
import Editor from "./components/Editor.vue";

export default {
  name: "App",

  components: {
    Editor,
  },

  data() {
    const that = this;
    return {
      url: "",
      selectedMethod: "",
      query: {},
      body: {},
      headers: {},
      loading: false,

      methods: [
        {
          method: "get",
          color: "green",
        },
        {
          method: "post",
          color: "orange",
        },

        {
          method: "put",
          color: "brown",
        },
        {
          method: "delete",
          color: "red",
        },
        {
          method: "patch",
          color: "purple",
        },
      ],

      editors: {
        headers: {
          label: "headers",
          editorConfig: {
            mode: "code",
            onChangeText: that.update("headers"),
          },
        },
        query: {
          label: "query",
          editorConfig: {
            mode: "code",
            onChangeText: that.update("query"),
          },
        },
        body: {
          label: "body",
          editorConfig: {
            mode: "code",
            onChangeText: that.update("body"),
          },
        },
        responseHeaders: {
          label: "headers",
          editorConfig: {
            mode: "tree",
          },
        },
        responseBody: {
          label: "body",
          style: {
            height: "48vh",
          },
          editorConfig: {
            mode: "tree",
          },
        },
      },
    };
  },

  computed: {
    isOk() {
      return this.url !== "" && this.isURL(this.url) && this.method !== "";
    },
  },

  methods: {
    isURL(str) {
      let url;

      try {
        url = new URL(str);
      } catch (_) {
        return false;
      }

      return url.protocol === "http:" || url.protocol === "https:";
    },

    update(field) {
      return (value) => {
        try {
          let obj = {};
          if (value.indexOf("[") >= 0 || value.indexOf("{") >= 0) {
            obj = JSON.parse(value);
          }
          this.$data[field] = obj;
        } catch (e) {
          // DO NOTHING
        }
      };
    },

    async ajax() {
      try {
        this.loading = true;
        let queryString = "";
        if (this.query) {
          queryString = new URLSearchParams(this.query).toString();
        }
        const res = await fetch(this.url + queryString, {
          method: this.method,
          headers: this.headers,
          body: this.body ? JSON.stringify(this.body) : undefined,
        });

        const headers = {};

        for (let pair of res.headers.entries()) {
          headers[pair[0]] = pair[1];
        }

        this.$refs.responseHeaders.set(headers);
        this.$refs.responseBody.set(await res.json());
      } catch (e) {
        console.log(e);
        this.$refs.responseBody.set({
          fetchError: e + "",
        });
      } finally {
        this.loading = false;
      }
    },
  },

  watch: {
    selectedMethod(v) {
      if (v === "get" || v === "head") {
        this.body = undefined;
      }
    },
  },
};
</script>
