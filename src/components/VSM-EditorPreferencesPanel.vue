<template>
  <v-dialog
      v-model="dialog"
      persistent
      max-width="50%"
      max-height="900px"
  >
    <v-card
        max-height="80vh"
        height="80vh">
      <v-card-title>
        <span class="headline" v-text="'Editor Preferences :'"></span>
      </v-card-title>
      <v-card-text height="100%">
        <v-tabs vertical height="65vh">
          <v-tab>
            <v-icon left>
              mdi-eye-outline
            </v-icon>
            Appearance
          </v-tab>

          <v-tab-item >
            <v-card height="64vh" width="100%">
              <v-col cols="12" sm="12" md="12" xl="6" class="ml-5">
                <v-row>
                  <span class="mt-3 mr-5"> Theme : </span>
                  <v-select
                      v-if="currentPreferences!==null"
                      :items="listTheme"
                      v-model="currentPreferences.theme"
                      label="Select Theme"
                      solo
                  ></v-select>
                </v-row>

              </v-col>
            </v-card>
          </v-tab-item>
        </v-tabs>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn
            color="blue darken-1"
            text
            @click="hide"
        >
          Cancel
        </v-btn>
        <v-btn
            color="blue darken-1"
            text
            @click="save"
        >
          Save
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  name: "VSM-EngineCodeEditPanel",

  data () {
    return {
      dialog: false,
      currentPreferences : null,
      listTheme : ["light", "dark"]
    };
  },

  props:["preferences", "bus"],

  mounted() {
    this.bus.$on('showEditorPreferencesPanel', this.show);
    this.bus.$on('hideEditorPreferencesPanel', this.hide);
  },

  methods:{
    save(){
      this.$emit("save", this.currentPreferences);
      this.hide();
    },
    show(){
      if(this.preferences=== null) return;
      this.currentPreferences = JSON.parse(JSON.stringify(this.preferences));
      this.dialog = true;
    },
    hide(){
      this.dialog = false;
    },
  },
}
</script>

<style scoped>

</style>