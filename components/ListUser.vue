<template>
  <div class="container pt-4">
    <div class="columns">
      <div class="column is-12">
        <label class="is-size-5 has-text-weight-bold is-pulled-left"
          >List User</label
        >
      </div>
    </div>
    <div class="columns">
      <table id="table-user" class="table is-hoverable is-fullwidth">
        <thead>
          <tr>
            <th>ID</th>
            <th>Avatar</th>
            <th>Email</th>
            <th>FirstName</th>
            <th>LastName</th>
            <th></th>
            <th></th>
          </tr>
        </thead>
      </table>
    </div>
    <div>
      <div v-if="openEditModal" class="modal is-active">
        <div class="modal-background"></div>
        <div class="modal-card">
          <header class="modal-card-head">
            <p class="modal-card-title">Update User</p>
            <button
              class="delete"
              aria-label="close"
              v-on:click="openEditModal = !openEditModal"
            ></button>
          </header>
          <section class="modal-card-body has-text-left">
            <div class="control">
              <label class="title is-6">FirstName:</label>&nbsp;&nbsp;
              <input
                class="input is-small"
                type="text"
                placeholder="FirstName"
                v-model="firstName"
              />
            </div>
            <br />
            <div class="control">
              <label class="title is-6">LastName:</label>&nbsp;&nbsp;
              <input
                class="input is-small"
                type="text"
                placeholder="LastName"
                v-model="lastName"
              />
            </div>
          </section>
          <footer class="modal-card-foot">
            <button class="button is-success" v-on:click="updateUser">
              Save changes
            </button>
            <button class="button" v-on:click="openEditModal = !openEditModal">
              Cancel
            </button>
          </footer>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const baseURL = "https://reqres.in/api";

export default {
  name: "ListUser",
  data() {
    return {
      tableUser: null,
      data: [],
      id: "",
      firstName: "",
      lastName: "",
      openEditModal: false,
    };
  },
  methods: {
    async getUsers() {
      try {
        const res = await fetch(`${baseURL}/users`);
        if (!res.ok) {
          const message = `An error has occured: ${res.status} - ${res.statusText}`;
          throw new Error(message);
        }
        const data = await res.json();
        this.data = data.data;
        this.tableUser.clear().rows.add(this.data).draw();

      } catch (e) {
        throw new Error(e.message);
      }
    },
    async updateUser() {
      try {
        const user = {
          first_name: this.firstName,
          last_name: this.lastName,
        };
        const res = await fetch(`${baseURL}/users/${this.id}`, {
          method: "PUT",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify(user),
        });

        if (!res.ok) {
          const message = `An error has occured: ${res.status} - ${res.statusText}`;
          throw new Error(message);
        }

        const data = await res.json();
        this.data.find((e) => e.id == this.id).first_name = data.first_name;
        this.data.find((e) => e.id == this.id).last_name = data.last_name;
        alert("Update user success!");
        this.openEditModal = !this.openEditModal;
        this.tableUser.clear().rows.add(this.data).draw();
      } catch (e) {
        throw new Error(e.message);
      }
    },
    async deleteUser() {
      try {
        const res = await fetch(`${baseURL}/users/${this.id}`, {
          method: "DELETE",
        });

        if (!res.ok) {
          const message = `An error has occured: ${res.status} - ${res.statusText}`;
          throw new Error(message);
        }

        const userIndex = this.data.findIndex((e) => e.id == this.id);
        this.data.splice(userIndex, 1);
        alert("Delete user success!");
        this.tableUser.clear().rows.add(this.data).draw();
      } catch (e) {
        throw new Error(e.message);
      }
    },
  },
  mounted() {
    this.tableUser = $("#table-user").DataTable({
      data: this.data,
      columns: [
        { data: "id" },
        {
          data: null,
          orderable: false,
          render: (data) => {
            return (
              '<figure class="image is-48x48"><img class="is-rounded" src="' +
              data.avatar +
              '"></figure>'
            );
          },
        },
        { data: "email" },
        { data: "first_name" },
        { data: "last_name" },
        {
          data: null,
          orderable: false,
          render: (data) => {
            return (
              '<button class="button is-warning is-small" onclick="window.parent.onOpenEditModal(' +
              data.id +
              ')">Edit</button>'
            );
          },
        },
        {
          data: null,
          orderable: false,
          render: (data) => {
            return (
              '<button class="button is-danger is-small" onclick="window.parent.confirmDeleteUser(' +
              data.id +
              ')">Delete</button>'
            );
          },
        },
      ],
    });

    this.getUsers();

    window.onOpenEditModal = (id) => {
      this.openEditModal = true;
      this.id = id;
      this.firstName = this.data.find((e) => e.id == id).first_name;
      this.lastName = this.data.find((e) => e.id == id).last_name;
    };

    window.confirmDeleteUser = (id) => {
      this.id = id;
      if (confirm("Do you want to delete this user?")) {
        this.deleteUser();
      }
    };
  },
};
</script>

<style>
.dataTables_length {
  display: none;
}

.dataTables_filter {
  margin-top: 20px;
  margin-bottom: 20px;
  text-align: right;
}

.input {
  width: 200px !important;
}

.pagination-link.is-current {
    color: white !important;
    border: 1px solid #42b883 !important;
    background-color: #42b883 !important;
}
</style>
