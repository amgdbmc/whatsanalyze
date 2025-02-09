<template>
  <div class="cta pa-2 pa-md-5 my-md-2">
    <div class="text-h4 text-md-h3 font-weight-bold pb-4">
      {{ $t("downloadPDF") }}
    </div>

    <v-row>
      <v-img
        class="ma-auto my-4"
        :src="require('@/assets/pdf-example.jpg')"
        max-height="147"
        max-width="250"
      />
    </v-row>

    <v-progress-linear
      v-show="isLoading"
      indeterminate
      color="blue"
      class="mb-2"
    ></v-progress-linear>

    <v-dialog v-model="showDownloadPopup" width="550">
      <template v-slot:activator="{ on, attrs }">
        <v-row justify="center" class="pa-0 ma-0">
          <v-col class="pa-0 ma-0">
            <v-btn elevation="10" @click="downloadSample">
              <v-icon class="mr-1">mdi-download</v-icon>
              <span><b>free</b> preview PDF</span>
            </v-btn>
            <v-col class="mt-2">
              <v-row justify="center" align="center">
                <b style="color: green">{{ 0 + " " + currency }}</b>
              </v-row>
            </v-col>
          </v-col>
          <v-col class="pa-0 ma-0">
            <v-btn
              class="white--text"
              dark
              v-bind="attrs"
              v-on="on"
              color="#07bc4c"
              style="max-width: 100%"
              elevation="10"
              @click="gtagEvent('full_pdf_pressed', GTAG_PAYMENT)"
            >
              <v-icon class="mr-1">mdi-download</v-icon>
              <span><b>full</b> chat PDF</span>
            </v-btn>
            <v-col class="mt-2">
              <v-row justify="center" align="center">
                <b style="color: green">{{ price + " " + currency }}</b>
                <span
                  class="px-1 ml-2"
                  style="color: white; background: red; border-radius: 5px"
                >
                  -60%</span
                >
              </v-row>
              <v-row justify="center" align="center">
                <s style="color: grey">{{ 4.99 + " " + currency }}</s>
              </v-row>
            </v-col>
          </v-col>
        </v-row>
      </template>
      <v-card>
        <v-card-title class="headline cyan" style="word-break: normal">
          <div class="text-h4 font-weight-bold">Nice !!</div>
          <span>You are just a step away from your PDF!</span>
        </v-card-title>
        <v-card-text class="pt-3 text-body-1 font-weight-bold">
          Supporting us keeps the 💻 running 🎉
        </v-card-text>
        <v-progress-linear
          v-show="isLoading"
          indeterminate
          color="blue"
          class="mb-2"
        ></v-progress-linear>
        <v-row cols="12" justify="center" align="center" class="pt-6 pr-10">
          <ChatVisualizationPayment
            @onCreateOrder="onCreateOrder"
            @onApprove="onApprove"
            @onError="onError"
            :currency="currency"
            :amount="price"
          />
        </v-row>
        <v-divider></v-divider>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="red darken-1" text @click="showDownloadPopup = false">
            Close
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>
<script>
import { render } from "~/functions/pdf";
import { GTAG_PAYMENT, GTAG_PDF, gtagEvent } from "~/functions/gtagValues";

export default {
  name: "PdfDownload",
  props: ["currency", "price", "chat", "attachments", "ego"],
  data() {
    return {
      showDownloadPopup: false,
      isLoading: false,
      GTAG_PAYMENT,
      GTAG_PDF,
    };
  },
  methods: {
    download() {
      gtagEvent("full_download", GTAG_PDF, 3);
      this.isLoading = true;
      render(this.chat, this.attachments, this.ego, false).then(
        () => (this.isLoading = false)
      );
    },
    onCreateOrder() {
      gtagEvent("created", GTAG_PAYMENT, 0);
    },
    onApprove() {
      gtagEvent("approved", GTAG_PAYMENT, 10);
      this.download();
    },
    onError() {},
    downloadSample() {
      gtagEvent("sample_download", GTAG_PDF, 2);
      this.isLoading = true;
      // download sample
      console.log(this.$route.query);
      render(
        this.chat,
        this.attachments,
        this.ego,
        !this.$route.query.free
      ).then(() => (this.isLoading = false));
    },
    gtagEvent,
  },
};
</script>
