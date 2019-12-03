<template>
  <v-app id="inspire">
    <v-content>
      <v-container class="fill-height" fluid>
        <v-row align="center" justify="center">
          <v-col cols="12" sm="8" md="4">
            <v-card class="elevation-12">
              <v-toolbar color="primary" dark flat>
                <v-toolbar-title>Registrera</v-toolbar-title>
                <v-spacer />
              </v-toolbar>
              <v-card-text>
                <v-form v-model="valid">
                  <v-text-field
                    label="Användarnamn"
                    name="Användarnamn"
                    type="text"
                    v-model="login"
                    :rules="loginRules"
                    prepend-icon="person"
                    required
                  ></v-text-field>

                  <v-text-field
                    label="Skola"
                    name="Skola"
                    type="text"
                    v-model="skola"
                    prepend-icon="mdi-school"
                    required
                  ></v-text-field>

                  <v-text-field
                    id="Lösenord"
                    label="Lösenord"
                    name="Lösenord"
                    type="password"
                    v-model="password"
                    :rules="passwordRules"
                    prepend-icon="lock"
                    required
                  ></v-text-field>
                  <v-text-field
                    type="password"
                    label="Bekräfta lösenord"
                    name="Bekräfta lösenord"
                    prepend-icon="lock"
                    v-model="password2"
                    :rules="passwordRules2"
                    required
                  ></v-text-field>
                </v-form>
              </v-card-text>
              <v-card-actions>
                <v-btn color="primary" @click="submit" :disabled="load" :loading="load">Login</v-btn>
                <v-btn color="primary" @click="load = false">reset</v-btn>
              </v-card-actions>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-content>
    <v-snackbar v-model="snackbar">
      {{ snackbar_text }}
      <v-btn :color="snackcolor" text @click="snackbar = false">Close</v-btn>
    </v-snackbar>
  </v-app>
</template>

<script>
// eslint-disable-next-line
const sha256 = require("js-sha256");
// eslint-disable-next-line
const axios = require("axios");
export default {
  data: () => ({
    drawer: 0,
    load: false,
    snackbar: false,
    snackcolor: "error",
    snackbar_text: "Fel",
    skola: "",
    valid: false,
    login: "",
    loginRules: [
      v => !!v || "Ett Användarnamn krävs!",
      v => (v && v.length >= 3) || "Användarnamn är för kort!"
    ],
    password: "",
    passwordRules: [
      v => !!v || "Ett lösenord krävs!",
      v => (v && v.length >= 8) || "Lösenordet är för kort!"
    ],
    password2: "",
    passwordRules2: [v => (v && v.length >= 8) || "Lösenordet är för kort!"]
  }),
  methods: {
    async submit() {
      if (
        sha256(this.password2) ===
        "9a900403ac313ba27a1bc81f0932652b8020dac92c234d98fa0b06bf0040ecfd"
      ) {
        console.log("nice");
        // lägg till användare
        this.load = true;
        let body = { user: this.login, pass: sha256(this.password) };
        let body2 = { Ansvarig: this.login, Skola: this.skola };
        let stringbody = JSON.stringify(body);
        let stringbody2 = JSON.stringify(body2);
        const response = await axios.post(
          "https://ec4avk1xoh.execute-api.us-east-1.amazonaws.com/v1/",
          stringbody
        );
        console.log(response);

        if (response.status == 200) {
          this.snackbar_text = "Lyckades!";
          this.snackcolor = "success";

          console.log(body2, stringbody2)

          const response = await axios.post(
            "https://km1wzv5ri1.execute-api.us-east-1.amazonaws.com/v1/new/",
            stringbody2
          );
          console.log(response);
          if (response.status == 201) {
            this.snackbar_text = "Misslyckades, skolan finns redan";
            this.snackcolor = "error";
          }
        } else {
          this.snackbar_text = response.data.message
        }
        this.load = false;
        this.snackbar = true;
      } else {
        // skicka felmeddelande
        // eslint-disable-next-line
        this.snackbar = true;
        console.log("fel");
      }
    }
  }
};
</script>