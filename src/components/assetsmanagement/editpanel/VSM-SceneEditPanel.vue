<template>
  <v-dialog
      v-model="dialog"
      persistent
      max-width="800px"
  >
    <v-card>
      <v-card-title>
        <span class="headline" v-text="(editionMode ? 'Edit Scene' : 'New Scene')"></span>
      </v-card-title>
      <v-card-text>
        <v-container>
              <v-text-field
                  v-if="currentScene!=null"
                  label="Name"
                  :rules="[rules.required, rules.counter, rules.existCharName]"
                  v-model="currentScene.name"
                  :value="currentScene.name"
              ></v-text-field>
                <v-file-input
                    class="mt-6"
                    @change="onChangeImage()"
                    @click="onClickFileInput()"
                    accept="image/*"
                    label="Default Image"
                    v-model="baseImage"
                    :clearable="false"
                ></v-file-input>
          <v-row>

            <v-spacer></v-spacer>

            <v-img
                v-if="baseImage!=null && baseImage.path!=''"
                :height="maxHeightImage"
                :src="baseImage.path"
                contain
                alt="Default Image"
            ></v-img>

            <img
                v-else
                :height="maxHeightImage"
                :src="require('../../../assets/logo.png')"
            >

            <v-spacer></v-spacer>

          </v-row>


        </v-container>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn
            color="blue darken-1"
            text
            @click="cancel"
        >
          Cancel
        </v-btn>
        <v-btn
            :disabled=!canSave
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
import jsonBaseCharacter from '../../../assets/base_characters.json';
import {readFileSync, writeFile, renameFile, deleteFile, getDate, removeDependencyVariableAsset} from '../../../lib.js';
import {mix_editassetpanel} from "@/mixins/MIX_EditAssetPanel";

const baseScene = jsonBaseCharacter;

export default {
  name: "VSM-SceneEditPanel",

  props: {
    height: {required: true},
  },

  mixins: [mix_editassetpanel],

  data () {
    return {
      type:"SceneEditPanel",
      baseImage: null,
      currentScene: null,
      rules: {
        required: value => !!value || 'Required.',
        counter: value => value.length <= 20 || 'Max 20 characters',
        existCharName: value => (this.assets[1].content.filter(e => e.name === value).length < 1 || this.previousName == value) || 'Already Exist',
      },
      oldimageinput: {name: "", path: ""},
      filesToDelete : [],
    };
  },

  computed: {
    canSave: function () {
      if(this.editionMode) return (this.currentScene!=null && this.currentScene.name != "" && (this.baseImage!=null) && ((!this.assets[1].content.some(a => a.name == this.currentScene.name)) || this.currentScene.name == this.previousName))
      return (this.currentScene!=null && this.currentScene.name != "" && this.baseImage!=null && !this.assets[1].content.some(a => a.name == this.currentScene.name));
    },
    maxHeightImage: function () {
      return this.height * 0.4;
    }
  },

  methods : {
    show(){
      this.currentScene = null;
      this.filesToDelete = [];
      if(this.editionMode){
        this.previousName = this.assets[1].content[this.indexEdition].name;
        this.currentScene = JSON.parse(JSON.stringify(this.assets[1].content[this.indexEdition]));
        this.baseImage = { name: this.currentScene.img, path: this.projectProp.directory + "Assets\\Scenes\\"+this.currentScene.img};
      } else {
        this.previousName = "";
        this.currentScene = JSON.parse(JSON.stringify(baseScene));
        this.baseImage = null;
      }
      this.dialog = true;
    },
    save(){
      if(this.canSave) {
        var filename = "";
        var filedata = "";
        if(this.editionMode){
          if(this.currentScene.name != this.previousName){
            if(this.baseImage.name == this.currentScene.img){
              filename = this.currentScene.name + getDate() + "." + this.currentScene.img.split('.').pop();
              renameFile(this.projectProp.directory + "Assets\\Scenes\\" + this.currentScene.img, this.projectProp.directory + "Assets\\Scenes\\" + filename);
            }

            removeDependencyVariableAsset("Scene", this.previousName, this.currentScene.name, this.assets, this.listPages);
          }

          if(this.baseImage.name != this.currentScene.img){
            deleteFile(this.projectProp.directory + "Assets\\Scenes\\" + this.currentScene.img);
            filename = this.currentScene.name + getDate() + "." + this.baseImage.name.split('.').pop();
            filedata = readFileSync(this.baseImage.path);
            writeFile(this.projectProp.directory + "Assets\\Scenes\\" + filename, filedata);
            this.currentScene.img = filename;
          } else if (filename != "") {
            this.currentScene.img = filename;
          }

          this.assets[1].content[this.indexEdition] = this.currentScene;

        } else {
          filename = this.currentScene.name + getDate() + "." + this.baseImage.name.split('.').pop();
          filedata = readFileSync(this.baseImage.path);
          writeFile(this.projectProp.directory + "Assets\\Scenes\\" + filename, filedata);

          this.currentScene.img = filename;

          this.assets[1].content.push(this.currentScene);
        }
        this.hide();
        this.$emit("accept");
      }
    },
    onChangeImage() {
      if(this.baseImage==null) this.baseImage = this.oldimageinput;
    },
    onClickFileInput() {
      this.oldimageinput = this.baseImage;
    }
  },

}
</script>

<style scoped>

</style>