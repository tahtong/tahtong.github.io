<script>
const fs = require("fs");
const { google } = require('googleapis');
const { GoogleAuth } = require('google-auth-library');

export default defineComponent({
  name: "Test",
});
// const { GoogleSpreadsheet } = require("google-spreadsheet");
// const creds = require("@/credentials.json");
// export default {
//   name: "Sheet",
//   //   data() {
//   //     return {
//   //       rows: [],
//   //       loading: true,
//   //     };
//   //   },
//   methods: {
//     async accessSpreadSheet() {
//       const doc = new GoogleSpreadsheet(
//         "1UJNpVSEYR_kNWIbNTssxBsWeIwECwjHtOOzmMlVjNsQ"
//       );
//       await doc.useServiceAccountAuth(creds);
//       await doc.loadInfo();
//       const sheet = doc.sheetsByIndex[0];
//       const rows = await sheet.getRows({
//         offset: 1,
//       });
//       console.log(rows);
//       //   this.rows = rows;
//     },
//   },
//   created() {
//     this.accessSpreadSheet();
//   },
// };
</script>