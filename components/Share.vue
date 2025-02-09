<template>
  <v-container ref="afd" style="position: relative">
    <slot ref="slot2"></slot>
    <v-btn
      @click="share"
      color="#016f94"
      fab
      dark
      style="position: absolute; right: 0; top: 0"
      :loading="loading"
      data-html2canvas-ignore
    >
      <v-icon size="35" v-if="canShare">mdi-share</v-icon>
      <v-icon size="35" v-else>mdi-download</v-icon>
    </v-btn>
  </v-container>
</template>

<script>
import { downloadBase64File } from "~/functions/utils";
import html2canvas from "html2canvas";
import { GTAG_RESULTS, gtagEvent } from "~/functions/gtagValues";

export default {
  name: "Share",
  props: {
    imageName: {
      type: String,
      default: "whatsanalyze.png",
    },
    useHtml2Canvas: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      canShare:
        (navigator.share &&
          navigator.canShare &&
          navigator.canShare({
            files: [new File([], "image.png", { type: "image/png" })],
          })) ||
        false,
      loading: false,
    };
  },
  methods: {
    async getCanvas() {
      if (this.useHtml2Canvas) {
        // extremly ugly but could not find a way around this
        // scrollY somehow is ignored in the html2canvas options
        let currScroll = window.scrollY;

        let offset = 0;
        if (this.$vuetify.breakpoint.xsOnly) {
          offset = 340;
        } else if (this.$vuetify.breakpoint.smOnly) {
          offset = 280;
        } else if (this.$vuetify.breakpoint.mdAndUp) {
          offset = 260;
        }
        window.scrollTo(0, offset);
        let html = this.$slots.default[0].child.$el;
        let removedTag = false;
        if (html.getAttribute("data-html2canvas-ignore") !== undefined) {
          html.removeAttribute("data-html2canvas-ignore");
          removedTag = true;
        }
        return html2canvas(html).then((_) => {
          window.scrollTo(0, currScroll);
          if (removedTag) html.setAttribute("data-html2canvas-ignore", "");
          return _;
        });
      } else {
        return this.$slots.default[0].child.$refs.canvas;
      }
    },
    async share() {
      let chartName = this.$slots.default[0].componentOptions.tag;
      this.loading = true;
      let canvas = await this.getCanvas();
      this.loading = false;

      if (this.canShare) {
        gtagEvent("share_" + chartName + "_pressed", GTAG_RESULTS, 0);

        canvas.toBlob((blob) => {
          navigator
            .share({
              title: "WhatsAnalze.com",
              text: this.$t("haveALook"),
              files: [
                new File([blob], chartName + "-" + this.imageName, {
                  type: "image/png",
                }),
              ],
            })
            .then(() => {
              gtagEvent("share_" + chartName + "_shared", GTAG_RESULTS, 2);
            });
        });
      } else {
        downloadBase64File(
          canvas.toDataURL(),
          chartName + "-" + this.imageName
        );

        gtagEvent("download_" + chartName, GTAG_RESULTS);
      }
    },
  },
};
</script>

<style scoped></style>
