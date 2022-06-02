<template>
  <div class="hello">
    <button v-on:click="launchVisu">visualise</button>
    <button v-on:click="closeVisu">close</button>
  </div>
</template>

<script>
import vtkWSLinkClient from "@kitware/vtk.js/IO/Core/WSLinkClient";
import SmartConnect from "wslink/src/SmartConnect";
import vtkRemoteView, {
  connectImageStream,
} from "@kitware/vtk.js/Rendering/Misc/RemoteView";

export default {
  name: "HelloWorld",

  props: {
    msg: String,
  },

  methods: {
    launchVisu() {
      vtkWSLinkClient.setSmartConnectClass(SmartConnect);

      this.divRenderer = document.createElement("div");
      document.body.appendChild(this.divRenderer);

      this.divRenderer.style.position = "relative";
      this.divRenderer.style.width = "100vw";
      this.divRenderer.style.height = "100vh";
      this.divRenderer.style.overflow = "hidden";

      this.remoteView = vtkRemoteView.newInstance({
        rpcWheelEvent: "viewport.mouse.zoom.wheel",
      });
      this.remoteView.setContainer(this.divRenderer);
      this.remoteView.setInteractiveRatio(0.7); // the scaled image compared to the clients view resolution
      this.remoteView.setInteractiveQuality(50); // jpeg quality

      window.addEventListener("resize", this.remoteView.resize);

      this.clientToConnect = vtkWSLinkClient.newInstance();

      // Error
      this.clientToConnect.onConnectionError((httpReq) => {
        const message =
          (httpReq && httpReq.response && httpReq.response.error) ||
          `Connection error`;
        console.error(message);
        console.log(httpReq);
      });

      // Close
      this.clientToConnect.onConnectionClose((httpReq) => {
        const message =
          (httpReq && httpReq.response && httpReq.response.error) ||
          `Connection close`;
        console.error(message);
        console.log(httpReq);
      });

      // hint: if you use the launcher.py and ws-proxy just leave out sessionURL
      // (it will be provided by the launcher)
      const config = {
        application: "cone",
        sessionURL: "ws://localhost:1234/ws",
      };

      // Connect
      this.clientToConnect
        .connect(config)
        .then((validClient) => {
          console.log("Client did connect");
          connectImageStream(validClient.getConnection().getSession());

          const session = validClient.getConnection().getSession();
          this.remoteView.setSession(session);
          this.remoteView.setViewId(-1);
          this.remoteView.render();
        })
        .catch((error) => {
          console.error(error);
        });
    },

    closeVisu() {
      this.remoteView?.delete();
      this.remoteView = null;

      this.clientToConnect?.disconnect(-1);
      this.clientToConnect = null;

      if (this.divRenderer) {
        document.body.removeChild(this.divRenderer);
        this.divRenderer = null;
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
