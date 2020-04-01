<template>
  <v-col>
    <v-data-table
      :headers="headers"
      :items="tutors"
      class="elevation-1"
      item-key="name"
      :search="search"
      dense
    >
      <template v-slot:top>
        <v-toolbar flat color="white">
          <v-toolbar-title> <h2>Verify Tutor</h2></v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
        </v-toolbar>
        <v-text-field
          v-model="search"
          label="Search"
          class="mx-4"
        ></v-text-field>
      </template>
      <template v-slot:item.verified="{ item }">
        <v-chip
          v-if="item.verified"
          class="ma-2"
          color="primary"
          dark
          outlined
          pill
        >
          Verified
          <v-icon right>mdi-account-outline</v-icon>
        </v-chip>
        <v-chip
          v-if="!item.verified"
          class="ma-2"
          color="error accent-4"
          outlined
          pill
        >
          Unverified
          <v-icon right>mdi-account-outline</v-icon>
        </v-chip>
      </template>
      <template v-slot:item.action="{ item }">
        <v-btn
          v-if="!item.verified"
          color="primary"
          small
          dark
          @click="verifyTutor(item)"
          >Verify</v-btn
        >
        <v-btn v-else color="secondary" small @click="unverifyTutor(item)"
          >Unverify</v-btn
        >
        <v-btn
          small
          :loading="false"
          :disabled="false"
          color="blue-grey"
          class="ma-2 white--text"
          fab
          @click="download(item)"
        >
          <v-icon dark>mdi-cloud-download</v-icon>
        </v-btn>
      </template>
      <template v-slot:no-data> </template>
    </v-data-table>
  </v-col>
</template>

<script lang="ts">
import { Vue, Component } from "vue-property-decorator";
import { Action, State } from "vuex-class";
import { VerifyActions, User } from "@/types";
import { VerifyRowItem } from "@/types";
import { SnackbarActions } from "@/types/snackbar";
import { AxiosResponse } from "axios";

@Component
export default class Verify extends Vue {
  @State(state => state.verify.isFetching) isFetching!: boolean;
  @State(state => state.verify.isSuccess) isSuccess!: boolean;
  @State(state => state.verify.isError) isError!: boolean;
  @State(state => state.verify.tutors) tutors!: Partial<User>[];

  @Action(VerifyActions.fetchTutors) fetchTutors!: Function;
  @Action(SnackbarActions.push) pushNewNotification!: Function;

  private search = "";
  private isloading = false;
  private editedIndex = -1;
  private editedItem = null;
  private headers = [
    {
      text: "Status",
      value: "verified",
      align: "center",
      class: "success--text title"
    },
    { text: "Username", value: "username" },
    { text: "Firstname", value: "firstName" },
    { text: "Surname", value: "lastName" },
    { text: "TimeStampSent", value: "evidenceSentDate" },
    { text: "Link", value: "evidenceInfo", align: "center" },
    { text: "Actions", value: "action", align: "center", sortable: false }
  ];

  mounted() {
    this.fetchTutors();
  }

  async verifyTutor(item: VerifyRowItem) {
    this.isloading = true;
    const response = await Vue.axios.post(`admin/verifyTutor/${item._id}`);
    if (response.status === 201) {
      if (response.data.verified) {
        item.verified = true;
        this.pushNotification("success", "Verify Succeeded");
        this.save(item);
      } else {
        this.pushNotification("error", "could not verify right now");
      }
    } else {
      this.pushNotification("error", "error occured");
    }
    this.isloading = false;
  }

  async unverifyTutor(item: VerifyRowItem) {
    this.isloading = true;
    const response = await Vue.axios.post(`admin/unverifyTutor/${item._id}`);
    if (response.status === 201) {
      if (!response.data.verified) {
        item.verified = false;
        this.pushNotification("success", "Unverify Succeeded");
        this.save(item);
      } else {
        this.pushNotification("error", "could not unverify right now");
      }
    } else {
      this.pushNotification("error", "error occured");
    }
    this.isloading = false;
  }

  save(item: VerifyRowItem) {
    this.editedIndex = this.tutors.indexOf(item);
    if (this.editedIndex > -1) {
      Object.assign(this.tutors[this.editedIndex], item);
    }
  }

  pushNotification(color: string, messages: string) {
    this.pushNewNotification({ color: color, message: messages });
  }

  // TODO: function to call to download file object
  forceFileDownload(response: AxiosResponse, item: VerifyRowItem) {
    const url = window.URL.createObjectURL(new Blob([response.data]));
    const link = document.createElement("a");
    link.href = url;
    link.setAttribute("download", item.evidenceInfo); //or any other extension
    document.body.appendChild(link);
    link.click();
  }

  // TODO: download function axios to retrive file
  download(item: VerifyRowItem) {
    this.$http({
      method: "get",
      url: "",
      responseType: "arraybuffer"
    }).then(response => {
      this.forceFileDownload(response, item);
    });
  }
}
</script>
<style lang="scss" scoped>
.container.fill-height {
  align-items: baseline;
}
</style>