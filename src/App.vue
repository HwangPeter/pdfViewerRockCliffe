<template>
  <v-app>
    <v-content class="lighten-4 grey">
      <PDFViewer/>
    </v-content>
  </v-app>
</template>
<script src="/__/firebase/7.14.3/firebase-app.js"></script>
<script src="/__/firebase/init.js"></script>

<script>
import PDFViewer from './components/PDFViewer';
import '@mdi/font/css/materialdesignicons.css'
import 'material-design-icons-iconfont/dist/material-design-icons.css'


export default {
  name: 'App',

  components: {
    PDFViewer,
  },
  icons: {
    iconfont: 'mdi', // default - only for display purposes
  },

  data: () => ({
    //
  }),
};
</script>