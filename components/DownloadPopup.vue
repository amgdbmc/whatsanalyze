<template>
  <v-col class="mb-8">
    <v-row justify="center">
      <div class="my-md-4 pa-8" :class="{ cta: !isSimple }">
        <div v-if="!isSimple" class="text-h3 font-weight-bold pb-4">
          Download all Graphs at once!
        </div>
        <div v-if="!isSimple" class="text-body-1 pb-2">
          Share them with your friends, all free just for you ❤️️
        </div>
        <v-dialog v-model="dialog" width="600">
          <template v-slot:activator="{ on }">
            <v-btn
              color="#07bc4c"
              dark
              v-on="on"
              @click="download"
              :loading="loading"
            >
              <v-icon class="mr-2">mdi-download</v-icon>Download Results
            </v-btn>
          </template>

          <v-card>
            <v-card-title class="headline cyan" style="word-break: normal">
              <div class="text-h4 font-weight-bold">{{ $t("didWeMake") }}</div>
              <span>{{ $t("buyUsCoffee") }}</span>
            </v-card-title>
            <v-card-text class="pt-3">
              <div>{{ $t("getResults") }}</div>
            </v-card-text>

            <v-row
              align="center"
              justify="center"
              class="mb-3"
              @click="paypalButtonPressed"
            >
              <form
                action="https://www.paypal.com/donate"
                method="post"
                target="_blank"
              >
                <input
                  type="hidden"
                  name="hosted_button_id"
                  value="EPCYG8WEF289G"
                />
                <input
                  type="image"
                  src="https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif"
                  border="0"
                  name="submit"
                  title="PayPal - The safer, easier way to pay online!"
                  alt="Donate with PayPal button"
                />
                <img
                  alt=""
                  border="0"
                  src="https://www.paypal.com/en_US/i/scr/pixel.gif"
                  width="1"
                  height="1"
                />
              </form>
            </v-row>
            <v-divider></v-divider>
            <v-progress-linear
              v-if="loading"
              indeterminate
              color="blue"
              class="mb-0"
            ></v-progress-linear>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="red darken-1" text @click="dialog = false">
                Close
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>

        <div v-if="!isSimple" class="text-text-h3 my-4">
          <v-col>
            <div v-if="!isSimple" class="text-body-1 pb-2">
              Looking for <b>PDF download</b>?
            </div>

            <v-btn
              v-if="!isSimple"
              color="#07bc4c"
              dark
              @click="
                gtagEvent('jump_to_pdf_download_cta', GTAG_INTERACTION, 0);
                $vuetify.goTo('#payButton', { duration: 300, offset: 100 });
              "
              ><v-icon class="mr-2">mdi-keyboard-return</v-icon
              >{{ $t("goToPDF") }}</v-btn
            >
          </v-col>
        </div>
      </div>
    </v-row>
  </v-col>
</template>

<script>
import html2canvas from "html2canvas";
import { downloadBase64File } from "~/functions/utils";
import {
  gtagEvent,
  GTAG_INTERACTION,
  GTAG_PAYMENT,
  GTAG_RESULTS,
} from "~/functions/gtagValues";

export default {
  name: "DownloadPopup",
  props: {
    chat: { type: Object },
    isSimple: { default: false, type: Boolean },
  },
  data() {
    return {
      dialog: false,
      loading: false,
      suffix: this.isSimple ? "-top" : "",
      GTAG_INTERACTION,
    };
  },
  methods: {
    download: function () {
      this.loading = true;
      gtagEvent("download_image", GTAG_RESULTS);

      setTimeout(() => {
        let additionalHeight = 0;
        document
          .querySelectorAll(".additional-height")
          .forEach((a) => (additionalHeight += a.clientHeight));

        let negativeHeight = 0;
        document
          .querySelectorAll("[remove-height-in-html2-canvas]")
          .forEach((a) => (negativeHeight -= a.clientHeight));

        let normalHeight = document.querySelector("#download-graphs")
          .clientHeight;

        //wordcloud
        let canvas = html2canvas(document.querySelector("#download-graphs"), {
          scrollX: 0,
          scrollY: -window.scrollY,
          height: normalHeight + additionalHeight + negativeHeight,
          onclone: function (clonedDoc) {
            let nonVisibleStuff = clonedDoc.querySelectorAll(
              ".only-visible-to-html2canvas"
            );
            nonVisibleStuff.forEach((y) => (y.style.display = "block"));
            return clonedDoc;
          },
        });

        let names = this.chat.messagesPerPerson
          .slice(0, 2)
          .map((person) => person.name)
          .join("-");
        canvas.then((canvas) => {
          downloadBase64File(
            canvas.toDataURL(),
            "whatsanlayze-results-" + names + ".png"
          );
          this.loading = false;
        });
      }, 250);
    },
    paypalButtonPressed() {
      gtagEvent("donation_download_results", GTAG_PAYMENT, 5);
    },
    gtagEvent,
  },
};
</script>
