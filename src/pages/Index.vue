<template>
  <q-page class="">
    <div class="q-pa-md">
      <h6>Для редактирования кликнете на ячейку</h6>
      <q-input
        filled
        bottom-slots
        v-model="text"
        type="search"
        hint=""
        label="Search name"
        counter
        maxlength="128"
        :dense="dense"
      >
        <template v-slot:append>
          <q-icon
            v-if="text !== ''"
            name="close"
            @click="text = ''"
            class="cursor-pointer"
          />
          <q-icon name="search" />
        </template>
      </q-input>

      <q-table
        title="Users"
        :data="users"
        :columns="columns"
        row-key="name"
        :loading="loading"
      >
        <template v-slot:body="props">
          <q-tr :props="props">
            
            <q-td
              key="name"
              :props="props"
              label-set="Save"
              label-cancel="Close"
            >
              {{ props.row.username }}
              <q-popup-edit
                v-model="props.row.username"
                buttons
                label-set="Save"
                label-cancel="Close"
                @save="updatingUser(props.row)"
                :validate="usernameValidation"
                @hide="usernameValidation"
              >
                <q-input
                  v-model="props.row.username"
                  hint="Введите от одной буквы до 128"
                  :error="errorUsername"
                  :error-message="errorMessageUsername"
                  dense
                  autofocus
                  counter
                />
              </q-popup-edit>
            </q-td>

            <q-td key="email" :props="props">
              {{ props.row.email }}
              <q-popup-edit
                v-model="props.row.email"
                buttons
                label-set="Save"
                label-cancel="Close"
                @save="updatingUser(props.row)"
                :validate="emailValidation"
                @hide="emailValidation"
              >
                <q-input
                  v-model="props.row.email"
                  dense
                  autofocus
                  counter
                  :error="errorEmail"
                  :error-message="errorMessageEmail"
                />
              </q-popup-edit>
            </q-td>

            <q-td key="address" :props="props">
              {{ props.row.address.street }}
              <q-popup-edit
                v-model="props.row.address.street"
                @save="updatingUser(props.row)"
              >
                <q-input
                  v-model="props.row.address.street"
                  dense
                  autofocus
                  counter
                />
              </q-popup-edit>
            </q-td>

            <q-td key="del" :props="props">
              <q-btn
                color="white"
                @click="deleteUser(props.row)"
                text-color="black"
                label="Delete"
              />
            </q-td>
          </q-tr>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script>
import * as _ from "lodash";
export default {
  name: "PageIndex",

  data() {
    return {
      loading: true,
      columns: [
        { name: "name", label: "name", field: "username" },
        { name: "email", label: "email", field: "email" },
        { name: "address", label: "address street", field: row => row.address.street},
        { name: "del", label: "Delete"}
      ],
      resUsers: [],
      users: [],
      errorUsername: false,
      errorMessageUsername: "",
      errorEmail: false,
      errorMessageEmail: "",
      text: "",
      dense: false,
    };
  },

  methods: {
    updatingUser(user) {
      this.$axios
        .put(`https://jsonplaceholder.typicode.com/users/${user.id}`)
        .then(userID => {
          console.log("put", userID);
        });
    },

    usernameValidation(name) {
      if (name === undefined) {
        return false;
      }

      if (name === "") {
        this.errorUsername = true;
        this.errorMessageUsername = "Обязательное поле";
        return false;
      }

      let nameLenght = name.length;

      if (nameLenght > 128) {
        this.errorUsername = true;
        this.errorMessageUsername = "Длина имени должна быть меньше 128";
        return false;
      }

      this.errorUsername = false;
      this.errorMessageUsername = "";
      return true;
    },

    emailValidation(email) {
      if (email === undefined) {
        return false;
      }
      if (email === "") {
        this.errorEmail = true;
        this.errorMessageEmail = "Обязательное поле";
        return false;
      }

      let emailLenght = email.length;

      if (emailLenght > 128) {
        this.errorEmail = true;
        this.errorMessageEmail ="Длина имени должна быть больше одной буквы и меньше 128 букв";
        return false;
      }

      if (email.indexOf("@") === -1) {
        this.errorEmail = true;
        this.errorMessageEmail = "Email должен содержать '@'";
        return false;
      }

      this.errorEmail = false;
      this.errorMessageEmail = "";
      return true;
    },

    deleteUser(user) {
      this.loading = true;
      this.$axios
        .delete(`https://jsonplaceholder.typicode.com/users/${user.id}`)
        .then(res => {
          this.loading = false;
          let cachUsers = _.cloneDeep(this.users)
          _.remove(cachUsers, { id: user.id } );
          _.remove(this.resUsers, { id: user.id } );
          this.users = cachUsers
        });
    },

  },

  watch: {
    text(val) {
      const re = new RegExp(val, "i");
      this.users = _.filter(this.resUsers, fild => fild.username.match(re));
    }
  },

  mounted() {
    this.$axios
        .get("https://jsonplaceholder.typicode.com/users")
        .then(users => {
          this.loading = false;
          this.users = _.cloneDeep(users.data);
          this.resUsers = _.cloneDeep(users.data);
        });
  }
};
</script>
