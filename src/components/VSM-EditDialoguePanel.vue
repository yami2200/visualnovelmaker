<template>
  <v-dialog
      v-model="dialog"
      persistent
      max-width="800px"
  >
    <v-card>
      <v-card-title>
        <v-tabs v-if="current != null" v-model="tab">
          <v-tab v-for="(tab,index) in current.tabs" :key="index"> {{ tab }} </v-tab>
        </v-tabs>
      </v-card-title>
      <v-card-text>
        <v-container>
          <v-tabs-items v-if="current != null" v-model="tab">
            <v-tab-item
                v-for="(tab,indexT) in current.tabs"
                :key="indexT"
            >
              <vsm-tabdialogue v-if="tab === 'Dialogue'" :current="current" :assets="assets"></vsm-tabdialogue>
              <vsm-tabcondition v-if="tab === 'Condition'" :current="current" :assets="assets"></vsm-tabcondition>
              <vsm-tabinput v-if="tab === 'Input'" :current="current" :assets="assets"></vsm-tabinput>
              <vsm-tabtransition v-if="tab === 'Transition'" :current="current" :assets="assets" :listPages="listPages"></vsm-tabtransition>
              <vsm-tabchoice v-if="tab === 'Choice'" :current="current" :assets="assets" :list-dialogues="listDialoguesTempo" :index-choice="index"></vsm-tabchoice>
              <vsm-tabscript v-if="tab === 'Script'" :current="current" :assets="assets"></vsm-tabscript>

            </v-tab-item>
          </v-tabs-items>
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
import baseTransitionArrival from "@/assets/base_dialoguetransition_arrival.json"
import TabDialogue from "@/components/dialogues/VSM-DialogueTabBasic"
import TabDialogueCondition from "@/components/dialogues/VSM-DialogueTabCondition"
import TabDialogueInput from "@/components/dialogues/VSM-DialogueTabInput"
import TabDialogueTransition from "@/components/dialogues/VSM-DialogueTabTransition"
import TabDialogueChoice from "@/components/dialogues/VSM-DialogueTabChoice"
import TabDialogueScript from "@/components/dialogues/VSM-DialogueTabScript"

export default {
  name: "VSM-EditDialoguePanel",

  props:["bus", "listDialogues", "listVar", "assets", "listPages"],

  components:{
    "vsm-tabdialogue" : TabDialogue,
    "vsm-tabcondition" : TabDialogueCondition,
    "vsm-tabinput" : TabDialogueInput,
    "vsm-tabtransition" : TabDialogueTransition,
    "vsm-tabchoice" : TabDialogueChoice,
    "vsm-tabscript" : TabDialogueScript
  },

  computed:{
    canSave(){
      return true;
    }
  },

  data: () => ({
    dialog: false,
    current: null,
    listDialoguesTempo : null,
    tab: "",
    index: -1
  }),

  methods:{
    save(){
      // ########################## CASE TRANSITION DIALOGUE
      if(this.current.transitionpage !== undefined){

        var p = this.listPages.filter((p) => p.title === this.current.transitionpage);

        if(this.current.transitionpage !== this.listDialogues[this.index].transitionpage){
          var transitionArrival = JSON.parse(JSON.stringify(baseTransitionArrival));
          transitionArrival.id = this.current.id;
          transitionArrival.title = this.current.title;
          if(p.length>0){
            if(this.listDialogues[this.index].transitionpage === ""){
              p[0].listDialogues.push(transitionArrival);
            } else {
              this.$emit("DeleteTransition", {page : this.listDialogues[this.index].transitionpage, id: this.current.id});
              p[0].listDialogues.push(transitionArrival);
            }


          }
        }

        if(this.current.title !== this.listDialogues[this.index].title){
          if(p.length>0){
            p[0].listDialogues.forEach((d) => {
              if(d.id !== undefined && d.id === this.current.id){
                d.title = this.current.title;
              }
            });
          }
        }
      }

      if(this.current.type === "choices"){
        for(var i = 0;i<this.listDialogues.length ; i++){
          this.listDialogues[i] = this.listDialoguesTempo[i];
        }
      }

      this.listDialogues[this.index] = this.current;
      this.hide();
      this.$emit("refresh", [this.index]);
    },
    cancel(){
      this.hide();
    },
    show(data){
      if(this.listDialogues[data.index] === undefined || this.listDialogues[data.index] === null || this.listDialogues[data.index].tabs.length === 0) return;

      this.current = null;
      this.listDialoguesTempo = JSON.parse(JSON.stringify(this.listDialogues));

      process.nextTick(() => {
        this.index = data.index;
        this.current = JSON.parse(JSON.stringify(this.listDialogues[data.index]));

        this.dialog = true;
      });
    },
    hide(){
      this.dialog = false;
    }
  },

  mounted() {
    this.bus.$on("showEditDialoguePanel", this.show);
  }
}
</script>

<style scoped>

</style>