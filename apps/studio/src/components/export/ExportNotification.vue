<template>
  <div />
</template>
<script>
import Noty from "noty";
import ExportInfo from "./mixins/export-info";
import { Export } from '../../lib/export';

export default {
  mixins: [ExportInfo],
  props: ['exporter'],
  data() {
    const exportName = this.exporter.table ? this.exporter.table.name : this.queryName;
    return {
      percentComplete: 0,
      notification: new Noty({
        text: `Exporting '${exportName}'`,
        layout: "bottomRight",
        timeout: false,
        closeWith: 'button',
        buttons: [
          Noty.button("Cancel", "btn btn-flat", this.cancelExport.bind(this)),
          Noty.button("Hide", "btn btn-primary", () => this.notification.close()),
        ],
        queue: "export",
      }),
    }
  },
  computed: {
    notificationText() {
      const exportName = this.exporter.table ? this.exporter.table.name : this.exporter.queryName;
      // CoPilot suggested the comment below, and it was right af lol
      // this is a hack to get the countExported to update
      const countExported = this.countExported;
      const percentComplete = this.percentComplete;
      return this.exporter.table
        ? `(${percentComplete}%) Exporting table '${exportName}'`
        : `(${countExported} rows) Exporting query '${exportName}'`
    },
  },
  methods: {
    cancelExport() {
      if (!this.exporter) {
        return;
      }
      this.exporter.abort();
      this.notification.close();
      const exportName = this.exporter.table ? this.exporter.table.name : this.exporter.queryName;
      this.$noty.error(`${exportName} export aborted`);
    },
    updateProgress(progress) {
      // not quite sure why this hackiness (and only this hackiness) finally made it work but i'll take it
      this.countExported = progress.countExported
      this.percentComplete = this.exporter.table ? progress.percentComplete : progress.countExported
    }
  },
  watch: {
    notificationText: {
      handler() {
        if (this.notification) {
          this.notification.setText(this.notificationText);
        }
      },
    },
  },
  mounted() {
    this.exporter.onProgress(this.updateProgress)
    this.notification.show();
  },
  beforeDestroy() {
    this.exporter.offProgress(this.updateProgress)
    this.notification.close();
  },
};
</script>
