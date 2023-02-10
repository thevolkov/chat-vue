<template>
  <v-layout column justify-center align-center>
    <v-flex xs12 sm8>
      <v-card min-width="400">
        <v-snackbar v-model="snackbar" :timeout="6000" top>
          {{ message }}
          <v-btn color="pink" flat @click="snackbar = false"> Закрыть </v-btn>
        </v-snackbar>
        <v-card-title>
          <h1 style="text-align: centr">ЧАТ by Volkov I.I.</h1>
        </v-card-title>
        <v-card-text>
          <v-form ref="form" v-model="valid" lazy-validation>
            <v-text-field
              v-model="name"
              :counter="16"
              :rules="nameRules"
              label="Имя юзера"
              required
            ></v-text-field>
            <v-text-field
              v-model="room"
              :rules="roomRules"
              label="Название комнаты"
              required
            ></v-text-field>
            <v-btn :disabled="!valid" color="primary" @click="submit">
              Войти
            </v-btn>
          </v-form>
        </v-card-text>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
import { mapMutations } from "vuex";
export default {
  layout: "empty",
  head: {
    title: "Войти",
  },
  sockets: {
    connect() {
      console.log("Socket connected");
    },
  },
  data: () => ({
    valid: true,
    snackbar: false,
    message: "",
    name: "",
    nameRules: [
      (v) => !!v || "Введите имя",
      (v) => (v && v.length <= 16) || "Имя не должно быть длиннее 16 символов",
    ],
    room: "",
    roomRules: [(v) => !!v || "Введите название комнаты"],
  }),
  mounted() {
    const { message } = this.$route.query;
    if (message === "noUser") {
      this.message = "Введите данные";
    } else if (message === "leftChat") {
      this.message = "Вы вышли из чата";
    }
    this.snackbar = !!this.message;
  },
  methods: {
    ...mapMutations(["setUser"]),
    submit() {
      if (this.$refs.form.validate()) {
        const user = {
          name: this.name,
          room: this.room,
        };

        this.$socket.emit("userJoined", user, (data) => {
          if (typeof data === "string") {
            console.log(data);
          } else {
            user.id = data.userId;
            this.setUser(user);
            this.$router.push("/chat");
          }
        });
      }
    },
  },
};
</script>

<style scoped>
button {
  margin: 10px 0px;
}
</style>
